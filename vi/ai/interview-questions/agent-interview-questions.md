---
title: Tổng hợp câu hỏi phỏng vấn AI Agent
description: Hệ thống tổng hợp các câu hỏi phỏng vấn AI Agent thường gặp, bao quát các trọng tâm như khái niệm cốt lõi của Agent, Agent Loop, Memory, Prompt Engineering, Context Engineering, MCP, Agent Skills, Harness Engineering, Workflow, Graph, Loop và kèm theo các bài viết tham khảo tương ứng.
category: AI
tag:
  - Phỏng vấn Agent
  - AI Agent
  - Phỏng vấn AI
head:
  - - meta
    - name: keywords
      content: Câu hỏi phỏng vấn AI Agent,Câu hỏi phỏng vấn Agent,phỏng vấn AI Agent,phỏng vấn Agent Loop,câu hỏi phỏng vấn Agent Memory,câu hỏi phỏng vấn MCP,câu hỏi phỏng vấn Prompt Engineering,phỏng vấn Context Engineering,phỏng vấn Harness Engineering,câu hỏi phỏng vấn Agent Skills
---

Sau khi nhận nhiệm vụ, Agent cần đọc context, quyết định action tiếp theo, gọi tool, quan sát kết quả, rồi phán đoán nên tiếp tục, kết thúc hay chuyển cho con người. Các câu hỏi phỏng vấn AI Agent về cơ bản đều xoay quanh execution chain này; Memory, MCP, Skills, Harness và Workflow cũng có thể được hiểu trong execution chain đó.

Các câu hỏi được nhóm theo các chương trong chuyên đề AI Agent của JavaGuide. Mỗi nhóm đều kèm bài viết chi tiết; ở đây chỉ tổng hợp các điểm kiến thức và câu hỏi, không lặp lại phần đáp án.

## Agent Basics

Nội dung liên quan: [《Khái niệm cốt lõi về AI Agent: Agent Loop, Plan-and-Execute, A2A, Agentic Workflows, đăng ký Tools》](../agent/agent-basis.md), [《Thiết kế hệ thống phối hợp Multi-Agent: phân chia nhiệm vụ, chia sẻ state, xử lý xung đột và khôi phục khi thất bại》](../agent/multi-agent.md)

Phần này thường bắt đầu từ định nghĩa Agent, sau đó hỏi tiếp về execution loop và cách orchestration. Khi chuẩn bị, bạn cần phân biệt được sự khác nhau giữa Chatbot, Workflow và Agent về task path, state và cách sử dụng tool.

Câu hỏi phỏng vấn thường gặp:

- AI Agent là gì? Khác gì với Chatbot thông thường?
- Hiểu công thức Agent = LLM + Planning + Memory + Tools như thế nào?
- Quy trình đầy đủ của Agent Loop là gì?
- Khác biệt cốt lõi giữa Agent với lập trình truyền thống và Workflow là gì?
- ReAct, Plan-and-Execute, Reflection, Multi-Agent lần lượt phù hợp với những trường hợp nào?
- Khi đăng ký Tools, tại sao description của tool lại rất quan trọng?
- Khi nào dùng Agent thuần túy, khi nào dùng Workflow hoặc Agentic Workflow?
- Những vấn đề chính của việc phối hợp Multi-Agent là gì? Tại sao trong production không thể tùy tiện sử dụng nhiều Agent?

![Kiến trúc cốt lõi AI Agent](https://oss.javaguide.cn/github/javaguide/ai/agent/agent-core-arch.png)

![Quy trình làm việc của Agent Loop](https://oss.javaguide.cn/github/javaguide/ai/agent/agent-loop-flow.png)

## Agent Memory

Nội dung liên quan: [《Hệ thống Memory của AI Agent: short-term memory, long-term memory và cơ chế tiến hóa memory》](../agent/agent-memory.md)

Câu hỏi về Memory sẽ đi sâu vào thông tin đến từ đâu, được lưu bao lâu, khi nào được đọc, cũng như cách xử lý khi thông tin hết hạn hoặc xung đột. Cần thảo luận riêng về lịch sử trò chuyện, state của task hiện tại và memory giữa các session.

Câu hỏi phỏng vấn thường gặp:

- Short-term memory và long-term memory của Agent khác nhau thế nào?
- Hệ thống memory của Agent cần giải quyết những vấn đề cốt lõi nào?
- Vector memory và Markdown memory lần lượt phù hợp với những trường hợp nào?
- Auto Memory là gì? Tại sao không thể tự động ghi vô hạn?
- Memory dùng chung của team nào phù hợp với Git và Code Review, memory nào phù hợp hơn với database?
- Nên xử lý việc nén memory, memory hết hạn và memory xung đột như thế nào?
- Làm thế nào để tránh long-term memory làm nhiễm context?
- Khi phỏng vấn, giải thích thế nào để cho thấy “có memory” không chỉ đơn giản là lưu lịch sử trò chuyện?

![Toàn cảnh phân loại memory của Agent](https://oss.javaguide.cn/github/javaguide/ai/agent/agent-memory-memory-taxonomy.svg)

## Prompt và Context Engineering

Nội dung liên quan: [《Prompt Engineering cho mô hình lớn là gì? Có những kỹ thuật prompt nào?》](../agent/prompt-engineering.md), [《Context Engineering là gì? Khác gì với Prompt Engineering?》](../agent/context-engineering.md)

Câu hỏi về Prompt tập trung vào cách diễn đạt instruction; câu hỏi về Context còn liên quan đến việc nạp historical state, kết quả tool, bằng chứng retrieval và task plan. Việc cắt, nén và cô lập context trong các task dài cũng là nội dung thường được hỏi thêm.

Câu hỏi phỏng vấn thường gặp:

- Prompt Engineering và Context Engineering khác nhau thế nào?
- Bốn yếu tố Role, Task, Context, Format của Prompt lần lượt giải quyết vấn đề gì?
- Few-Shot, CoT, phân rã task và structured output lần lượt phù hợp với những trường hợp nào?
- Tấn công prompt injection là gì? Có những cách phòng vệ thường gặp nào?
- Tại sao trong trường hợp Agent, chỉ tối ưu Prompt là chưa đủ?
- Context Engineering cần giải quyết những vấn đề nào?
- Static rule, dynamic information, kết quả tool và memory nên được đưa vào context như thế nào?
- Khi context của task dài bị tràn, lần lượt sử dụng Compaction, structured notes và Sub-agent như thế nào?

![Prompt engineering và context engineering](https://oss.javaguide.cn/github/javaguide/ai/context-engineering/context-engineering-vs-prompt-engineering.png)

## MCP và Agent Skills

Nội dung liên quan: [《Model Context Protocol (MCP) là gì? Có quan hệ thế nào với Function Calling và Agent?》](../agent/mcp.md), [《Agent Skills là gì? Rốt cuộc khác Prompt và MCP ở đâu?》](../agent/skills.md)

Function Calling, MCP và Skills nằm ở các khâu khác nhau: model cần biểu đạt ý định gọi, host cần tích hợp tool, còn Agent phải load workflow và tài liệu cần thiết để hoàn thành task. Nhóm câu hỏi này thường hỏi tiếp về permission, validation tham số, timeout và audit.

Câu hỏi phỏng vấn thường gặp:

- MCP giải quyết vấn đề gì? Tại sao thường được ví như USB-C trong lĩnh vực AI?
- MCP Client, MCP Server và Host lần lượt là gì?
- Tools, Resources và Prompts của MCP lần lượt giải quyết vấn đề gì?
- MCP khác Function Calling thế nào?
- MCP Server cấp production cần thực hiện những governance bảo mật nào?
- Agent Skills là gì? Ranh giới giữa nó với Prompt, MCP và Function Calling là gì?
- Tại sao Skills cần được load lazy?
- Làm routing cho Skill như thế nào? Tại sao nó tương tự RAG nhưng có mục tiêu khác?
- Những lỗi nào dễ mắc nhất khi viết một `SKILL.md`?

## Harness Engineering

Nội dung liên quan: [《Harness Engineering: framework kiểm tra sáu lớp, quản lý context và thực hành engineering》](../agent/harness-engineering.md)

Harness Engineering tập trung vào execution environment bên ngoài model, bao gồm task management, cung cấp context, phản hồi của tool, verification và error recovery. Các câu hỏi liên quan thường yêu cầu đưa những khái niệm trừu tượng này về các component cụ thể.

Câu hỏi phỏng vấn thường gặp:

- Harness Engineering là gì? Có quan hệ thế nào với Prompt Engineering và Context Engineering?
- Tại sao nói Agent = Model + Harness?
- Sáu lớp trong framework kiểm tra Harness do chuyên đề AI Agent của JavaGuide tổng hợp lần lượt giải quyết vấn đề gì?
- Sau khi năng lực model được nâng cấp, tại sao một số cơ chế trong Harness cần được verification lại?
- Context pollution, tích lũy code entropy và độ tin cậy khi gọi tool lần lượt được governance như thế nào?
- Tại sao engineering Agent cần evaluator, verifier và task state management?
- Khi các team tuyến đầu thực hiện engineering Agent, những khó khăn thường gặp chung là gì?

![Quan hệ giữa Harness với Prompt/Context Engineering](https://oss.javaguide.cn/github/javaguide/ai/harness/harness-engineering-layers-arch.png)

## Workflow, Graph và Loop

Nội dung liên quan: [《Workflow, Graph và Loop trong AI Workflow: từ khái niệm đến triển khai》](../agent/workflow-graph-loop.md)

Câu hỏi về workflow chủ yếu kiểm tra cấu trúc flow, lưu state và điều khiển loop. Ngoài việc giải thích Node, Edge và State, bạn còn cần chuẩn bị cho các vấn đề engineering như khôi phục sau khi interrupt, cập nhật song song và điều kiện dừng.

Câu hỏi phỏng vấn thường gặp:

- Tại sao hệ thống AI cần workflow?
- Quan hệ giữa Workflow, Graph và Loop là gì?
- Graph Loop khác Agent Loop thế nào?
- Làm thế nào để Loop tránh vòng lặp vô hạn?
- Nên chọn chiến lược cập nhật State thế nào? Replace, Append và Reducer lần lượt phù hợp với những field nào?
- Conditional edge khác dynamic routing thế nào?
- Khi gọi tool thất bại, lỗi nào phù hợp để retry? Xử lý authentication failure và non-idempotent write operation thế nào?
- Khôi phục workflow sau khi bị interrupt như thế nào?
- Workflow có những rủi ro bảo mật đặc thù nào?

## Câu hỏi thiết kế tổng hợp

Ngoài việc kiểm tra khái niệm, phỏng vấn dự án còn tiếp tục hỏi về architecture, cơ sở lựa chọn, bằng chứng hiệu quả và failure trên production. Có thể tham khảo [《Trình bày dự án Agent khi phỏng vấn thế nào? Từ system architecture, lựa chọn công nghệ đến tổng kết Badcase》](./agent-project-interview-guide.md) để biết cách tổ chức câu trả lời đầy đủ.

Câu hỏi tổng hợp sẽ đưa các component phía trên vào cùng một task, tập trung kiểm tra việc lựa chọn và xử lý sự cố:

- Nếu yêu cầu Agent hoàn thành một task dài cần gọi nhiều tool, bạn sẽ phân tách các bước thực thi như thế nào?
- Những node nào phù hợp để giao cho model phán đoán, node nào nên được rule hoặc code kiểm soát?
- Plan, kết quả tool và state trung gian của Agent được lưu như thế nào? Khôi phục sau khi bị interrupt ra sao?
- Khi tool có write operation, thiết kế permission, validation tham số, xác nhận lần hai và audit như thế nào?
- Khi nào cần Multi-Agent? Làm thế nào để kiểm soát communication cost và consistency của state?
- Bạn sẽ ghi lại những Trace nào, và dùng metric nào để đánh giá completion rate của task, việc gọi tool và execution trace?
