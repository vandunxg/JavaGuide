---
title: "Chuyên đề AI Agent: Agent Loop, Memory, Prompt, Context, MCP và Skills"
description: "Lộ trình học và phỏng vấn AI Agent, bao quát Agent Loop, Memory, Prompt Engineering, Context Engineering, MCP, Agent Skills, Harness Engineering và workflow AI."
category: AI
tag:
  - AI Agent
  - LLM
  - Phát triển ứng dụng AI
sidebar: false
---

<!-- @include: @small-advertisement.snippet.md -->

Agent không phải là “chatbot biết gọi Tools”. Khi nhiệm vụ trở nên dài hơn, nó phải xử lý state, Memory, permission, retry khi thất bại, cắt gọn context và ranh giới thực thi.

Chuyên đề **AI Agent** này dành cho developer muốn hiểu và triển khai ứng dụng Agent, xem Agent Loop, Memory, Prompt, Context, Tools, MCP, Skills, Harness Engineering và Workflow trên cùng một tuyến kỹ thuật.

## Dành cho ai

- Developer muốn hiểu nguyên lý và cách triển khai AI Agent.
- Engineer đang phát triển ứng dụng AI liên quan đến gọi Tools, nhiệm vụ tự động hóa, suy luận nhiều lượt và thực thi nhiệm vụ dài.
- Người chuẩn bị câu hỏi phỏng vấn về Agent, MCP, Prompt, Context, Skills và workflow.

## Trọng tâm học

- Sự khác nhau giữa Agent và Workflow không nằm ở việc có gọi LLM hay không, mà ở việc có vòng lặp khép kín gồm quan sát, lập kế hoạch, hành động và phản hồi hay không.
- Memory giải quyết việc lưu giữ thông tin giữa các lượt và nhiệm vụ, nhưng phải thiết kế vòng đời, ranh giới lưu trữ và quản trị quyền riêng tư.
- Prompt Engineering chú trọng hơn vào cách diễn đạt chỉ dẫn, còn Context Engineering chú trọng hơn vào việc đưa đúng thông tin vào context ở đúng thời điểm.
- MCP, Skills và Harness Engineering quyết định Agent có thể tích hợp ổn định, an toàn và có khả năng mở rộng với Tools và môi trường thực tế hay không.

## Thứ tự đọc đề xuất

1. [Khái niệm cốt lõi về AI Agent](./agent-basis.md): xây dựng nhận thức tổng thể về Agent trước.
2. [Prompt Engineering cho LLM](./prompt-engineering.md), [Context Engineering](./context-engineering.md): hiểu cách chỉ dẫn và context cùng tác động đến output.
3. [Hệ thống Memory của AI Agent](./agent-memory.md): bổ sung Memory ngắn hạn, Memory dài hạn và vòng đời Memory.
4. [Phân tích chi tiết MCP Protocol](./mcp.md), [Giải thích chi tiết Agent Skills](./skills.md): hiểu việc tích hợp Tools và mở rộng capability.
5. [Thiết kế hệ thống cộng tác nhiều Agent](./multi-agent.md): hiểu việc chia nhỏ nhiệm vụ, chia sẻ state, xung đột khi chạy đồng thời và khôi phục sau thất bại.
6. [Harness Engineering: framework kiểm tra sáu lớp, quản lý context và thực hành engineering](./harness-engineering.md), [Workflow, Graph và Loop trong workflow AI](./workflow-graph-loop.md), [Loop Engineering là gì](./loop-engineering.md): bước vào engineering Agent cấp production.

## Bài viết cốt lõi

- [Khái niệm cốt lõi về AI Agent](./agent-basis.md): hệ thống hóa quá trình phát triển của AI Agent, giải thích các khái niệm nền tảng như Agent Loop, Context Engineering và đăng ký Tools.
- [Hệ thống Memory của AI Agent](./agent-memory.md): tìm hiểu sâu thiết kế Memory ngắn hạn và dài hạn, nắm được hình thức lưu trữ, thao tác vòng đời và chiến lược tối ưu engineering cấp production.
- [Thiết kế hệ thống cộng tác nhiều Agent](./multi-agent.md): trình bày từ contract nhiệm vụ, state dùng chung và xử lý xung đột đến checkpoint, tiếp quản, bù trừ và can thiệp thủ công.
- [Prompt Engineering cho LLM](./prompt-engineering.md): trình bày từ bốn yếu tố của Prompt, các kỹ thuật thường gặp đến thực hành bảo mật cấp doanh nghiệp.
- [Context Engineering](./context-engineering.md): nắm các kỹ thuật then chốt như tổ chức rule tĩnh, gắn thông tin động và hạ cấp theo ngân sách Token.
- [Giải thích chi tiết Agent Skills](./skills.md): hiểu triết lý thiết kế Skills và khác biệt bản chất giữa Skills với Prompt, MCP và Function Calling.
- [Phân tích chi tiết MCP Protocol](./mcp.md): hiểu khái niệm cốt lõi, thiết kế architecture và best practice cấp production của MCP Protocol.
- [Harness Engineering: framework kiểm tra sáu lớp, quản lý context và thực hành engineering](./harness-engineering.md): phân tích cách các team như OpenAI, Anthropic và Stripe triển khai engineering Agent.
- [Workflow, Graph và Loop trong workflow AI](./workflow-graph-loop.md): so sánh khác biệt giữa workflow truyền thống và workflow AI, bao quát cách triển khai bằng Spring AI Alibaba và LangGraph.
- [Loop Engineering là gì? Vì sao nói đây là bình mới rượu cũ?](./loop-engineering.md): đặt Loop Engineering trong Agent Loop, Context, Harness, Skills, MCP và vòng lặp verification để hiểu điểm mới thực sự của nó.

## Câu hỏi thường gặp

- Agent khác Workflow, Chatbot và gọi Tools thông thường như thế nào?
- Quan sát, lập kế hoạch, hành động và tự phản tỉnh trong Agent Loop lần lượt phụ trách việc gì?
- Memory nên lưu gì, không nên lưu gì? Làm thế nào để tránh rủi ro nhiễm bẩn và quyền riêng tư?
- Vì sao không thể đánh đồng Prompt Engineering với Context Engineering?
- Ranh giới giữa MCP, Skills và Function Calling lần lượt nằm ở đâu?
- Agent thực hiện nhiệm vụ dài kiểm soát context, permission, retry khi thất bại và khả năng quan sát như thế nào?

## Chuyên đề liên quan

- [Hệ thống kiến thức về phát triển ứng dụng AI](../)
- [Chuyên đề nền tảng LLM](../llm-basis/)
- [Chuyên đề RAG](../rag/)
- [Chuyên đề câu hỏi phỏng vấn phát triển ứng dụng AI](../interview-questions/)
- [Trình bày dự án Agent trong phỏng vấn như thế nào?](../interview-questions/agent-project-interview-guide.md)

## Tổng kết

Các vấn đề engineering của Agent dần lộ rõ khi nhiệm vụ trở nên dài hơn: model phải ra quyết định theo state trong một loop, thông tin lịch sử cần được lưu trữ và truy xuất, Tools bên ngoài cần quy tắc tích hợp, permission và audit rõ ràng, còn thất bại phải có khả năng replay và xử lý.

Khi đọc, bạn có thể phân biệt trước trách nhiệm của Agent, Workflow và gọi Tools thông thường, sau đó bổ sung chi tiết triển khai theo chuỗi Prompt, Context, Memory, MCP, Skills và Harness. Khi làm dự án cụ thể, hãy bắt đầu từ vòng lặp khép kín tối thiểu có thể kiểm chứng, rồi chỉ thêm cơ chế khi thực tế xuất hiện vấn đề về context, chi phí, permission hoặc độ ổn định.

<!-- @include: @article-footer.snippet.md -->
