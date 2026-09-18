---
title: Chuyên đề câu hỏi phỏng vấn phát triển ứng dụng AI
description: Lộ trình ôn tập và câu hỏi phỏng vấn phát triển ứng dụng AI, tập trung vào nền tảng LLM, AI Agent, RAG và thiết kế hệ thống AI, tổng hợp các câu hỏi thường gặp, phù hợp với AI Engineer và người chuyển từ backend sang AI.
category: AI
tag:
  - AI
  - Phỏng vấn AI
  - Phỏng vấn backend
sidebar: false
---

<!-- @include: @small-advertisement.snippet.md -->

Phỏng vấn phát triển ứng dụng AI hiếm khi chỉ hỏi “khái niệm là gì”. Câu hỏi thường đi sâu từ một dự án: tại sao lại thiết kế như vậy, khi xảy ra sự cố thì kiểm tra thế nào, sau khi đưa vào production thì đánh giá ra sao, quản lý chi phí và bảo mật thế nào.

Chuyên đề **câu hỏi phỏng vấn phát triển ứng dụng AI** này dành cho AI Engineer, vị trí phát triển ứng dụng AI và người chuyển từ backend sang AI. Nội dung kết nối các câu hỏi về “nền tảng LLM, AI Agent, RAG và thiết kế hệ thống AI” thành một lộ trình ôn tập.

## Dành cho ai

- Người chuẩn bị phỏng vấn các vị trí phát triển ứng dụng AI, AI Engineer hoặc chuyển từ backend sang AI.
- Người đã biết một số khái niệm AI nhưng khi trả lời câu hỏi phỏng vấn thường trình bày lan man, hời hợt hoặc chỉ dừng ở mức Demo.
- Developer muốn sắp xếp kinh nghiệm dự án theo cách trình bày “vấn đề -> nguyên lý -> giải pháp -> đánh đổi -> triển khai”.

## Trọng tâm học

- Câu hỏi nền tảng LLM tập trung làm rõ Token, context, tham số sampling, structured output, model invocation và evaluation.
- Câu hỏi Agent tập trung làm rõ Agent Loop, Memory, Prompt, Context, MCP, Skills và workflow.
- Câu hỏi RAG tập trung làm rõ xử lý tài liệu, vector retrieval, hybrid retrieval, Rerank, GraphRAG, cập nhật knowledge base và evaluation.
- Câu hỏi thiết kế hệ thống tập trung làm rõ model gateway, observability, chi phí, bảo mật, gradual rollout và real-time voice Agent.

## Thứ tự đọc đề xuất

1. [Hướng dẫn phỏng vấn phát triển ứng dụng AI](./ai-interview-guide.md): đọc phần tổng quan trước để xây dựng bản đồ ôn tập tổng thể.
2. [Tổng hợp câu hỏi phỏng vấn nền tảng LLM](./llm-interview-questions.md): bổ sung các khái niệm nền tảng LLM và luồng gọi API.
3. [Tổng hợp câu hỏi phỏng vấn AI Agent](./agent-interview-questions.md): nắm các khái niệm thường gặp và vấn đề triển khai liên quan đến Agent.
4. [Trình bày dự án Agent khi phỏng vấn thế nào?](./agent-project-interview-guide.md): tổ chức khái niệm thành bối cảnh dự án, kiến trúc hệ thống, lựa chọn công nghệ, bằng chứng hiệu quả và retrospective Badcase.
5. [Tổng hợp câu hỏi phỏng vấn RAG](./rag-interview-questions.md): ôn tập các vấn đề retrieval, ranking, cập nhật và evaluation xoay quanh hỏi đáp knowledge base doanh nghiệp.
6. [Tổng hợp câu hỏi phỏng vấn thiết kế hệ thống AI](./ai-system-design-interview-questions.md): kết nối các module trên thành cách trình bày thiết kế hệ thống production.

## Bài viết cốt lõi

- [Hướng dẫn phỏng vấn phát triển ứng dụng AI](./ai-interview-guide.md): cổng vào tổng hợp các câu hỏi phỏng vấn phát triển ứng dụng AI, tổ chức lộ trình ôn tập theo nền tảng LLM, AI Agent, RAG và thiết kế hệ thống AI.
- [Tổng hợp câu hỏi phỏng vấn nền tảng LLM](./llm-interview-questions.md): bao quát Token, context window, tham số sampling, gọi API, structured output, Function Calling, MCP và evaluation ứng dụng AI.
- [Tổng hợp câu hỏi phỏng vấn AI Agent](./agent-interview-questions.md): bao quát các khái niệm cốt lõi của Agent, Memory, Prompt Engineering, Context Engineering, MCP, Agent Skills, Harness Engineering và AI workflow.
- [Trình bày dự án Agent khi phỏng vấn thế nào?](./agent-project-interview-guide.md): dùng một chuỗi bằng chứng có thể kiểm chứng để trình bày rõ kiến trúc, lựa chọn công nghệ, chỉ số và Badcase của dự án Agent, tránh chỉ liệt kê tên công nghệ.
- [Tổng hợp câu hỏi phỏng vấn RAG](./rag-interview-questions.md): bao quát nền tảng RAG, Embedding, vector database, chiến lược Chunk, xử lý tài liệu, tối ưu retrieval, GraphRAG, cập nhật knowledge base và RAG evaluation.
- [Tổng hợp câu hỏi phỏng vấn thiết kế hệ thống AI](./ai-system-design-interview-questions.md): bao quát kiến trúc production, model gateway, quản lý Prompt, observability, evaluation, quản trị bảo mật và real-time voice Agent.

## Câu hỏi thường gặp

- Khi interviewer hỏi “Bạn đã làm ứng dụng AI chưa?”, nên trả lời thế nào từ bối cảnh nghiệp vụ, kiến trúc, đánh giá hiệu quả đến quản trị khi đưa vào production?
- Tại sao khi gọi API của LLM cần retry, rate limiting, fallback và structured validation?
- Agent khác workflow thông thường ở điểm nào?
- Khi RAG không retrieval được, retrieval sai hoặc generation sai thì kiểm tra thế nào?
- Thiết kế hệ thống ứng dụng AI thể hiện stability, observability, kiểm soát chi phí và quản trị bảo mật thế nào?

## Chuyên đề liên quan

- [Hệ thống kiến thức về phát triển ứng dụng AI](../)
- [Chuyên đề nền tảng LLM](../llm-basis/)
- [Chuyên đề AI Agent](../agent/)
- [Chuyên đề RAG](../rag/)

<!-- @include: @article-footer.snippet.md -->
