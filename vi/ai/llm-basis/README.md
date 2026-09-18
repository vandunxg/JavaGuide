---
title: "Chuyên đề nền tảng LLM: cơ chế vận hành, gọi API, output có cấu trúc và đánh giá"
description: Nền tảng LLM cho phỏng vấn và lộ trình học, bao quát cơ chế vận hành LLM, thực hành engineering khi gọi API, output có cấu trúc, Function Calling, Tool Calling và đánh giá ứng dụng AI.
category: AI
tag:
  - LLM Basics
  - LLM
  - Phát triển ứng dụng AI
sidebar: false
---

<!-- @include: @small-advertisement.snippet.md -->

API của LLM trông chỉ như một request và một response, nhưng khi triển khai thực tế, mọi vấn đề đều ẩn trong chi tiết: Token được tiêu tốn thế nào, vì sao context window không đủ, tham số sampling có khiến câu trả lời dao động không, vì sao output có cấu trúc đôi khi thất bại, trước khi lên production làm sao chứng minh hiệu quả thực sự đã tốt hơn.

Chuyên đề **nền tảng LLM** này dành cho người mới bắt đầu phát triển ứng dụng AI và triển khai thực tế, trước hết giải thích rõ các vấn đề nền tảng này, sau đó học tiếp Agent, RAG và system design sẽ thuận lợi hơn.

## Dành cho ai

- Developer đang học các khái niệm nền tảng LLM và cách gọi API LLM.
- Engineer đã làm Prompt Demo nhưng chưa quen với production-grade call flow, output có cấu trúc và đánh giá.
- Người chuẩn bị cho các câu hỏi phỏng vấn về nền tảng LLM, Function Calling, Tool Calling và đánh giá ứng dụng AI.

## Trọng tâm học

- Token, context window, Temperature, Top P, stop word và các tham số khác ảnh hưởng đến output của model như thế nào.
- Việc gọi API LLM ở production cần xử lý streaming response, timeout, retry, rate limiting, fallback, log và audit.
- Output có cấu trúc không thể chỉ dựa vào Prompt, mà còn phải kết hợp JSON Schema, Function Calling, Tool Calling và validation phía server; ngay cả vậy vẫn phải chuẩn bị fallback khi thất bại.
- Đánh giá ứng dụng AI cần phân biệt Golden Set offline, LLM-as-Judge, canary online, Trace replay và regression liên tục.

## Thứ tự đọc đề xuất

1. [Cơ chế vận hành LLM: Token, context window và tham số sampling ảnh hưởng output thế nào](./llm-operation-mechanism.md): trước hết hiểu Token, context window và tham số sampling.
2. [Thực hành engineering khi gọi API LLM](./llm-api-engineering.md): tiếp theo xem cách đưa việc gọi LLM vào call flow backend thực tế.
3. [Giải thích chi tiết output có cấu trúc của LLM](./structured-output-function-calling.md): bổ sung nền tảng về response có cấu trúc và gọi tool.
4. [Hệ thống đánh giá ứng dụng AI](./llm-evaluation.md): cuối cùng xây dựng phương pháp đánh giá chất lượng và regression trước khi lên production.

## Bài viết cốt lõi

- [Cơ chế vận hành LLM: Token, context window và tham số sampling ảnh hưởng output thế nào](./llm-operation-mechanism.md): chuyển các khái niệm như Token, context window, Temperature thành các tham số engineering có thể quan sát và debug.
- [Thực hành engineering khi gọi API LLM](./llm-api-engineering.md): phân tích production call flow của ứng dụng AI khi gọi API LLM, bao quát streaming output, retry, rate limiting, response có cấu trúc và triển khai backend.
- [Giải thích chi tiết output có cấu trúc của LLM](./structured-output-function-calling.md): làm rõ JSON Schema, Function Calling, Tool Calling và MCP lần lượt đảm nhiệm gì trong một lần gọi tool.
- [Hệ thống đánh giá ứng dụng AI](./llm-evaluation.md): từ Golden Set, LLM-as-Judge, metric của RAG/Agent, Trace replay đến regression trong CI, giải thích cách nghiệm thu ứng dụng AI.

## Câu hỏi thường gặp

- Vì sao cùng một Prompt nhưng mỗi lần output lại khác nhau?
- Context window lớn hơn có nhất thiết tốt hơn không?
- Vì sao output có cấu trúc đôi khi thất bại? Làm fallback và validation thế nào?
- Function Calling, Tool Calling và MCP lần lượt giải quyết vấn đề gì?
- Trước khi đưa ứng dụng AI lên production, làm sao chứng minh “hiệu quả đã tốt hơn” thay vì chỉ dựa vào cảm nhận?

## Chuyên đề liên quan

- [Hệ thống kiến thức phát triển ứng dụng AI](../)
- [Chuyên đề AI Agent](../agent/)
- [Chuyên đề RAG](../rag/)
- [Chuyên đề câu hỏi phỏng vấn phát triển ứng dụng AI](../interview-questions/)

<!-- @include: @article-footer.snippet.md -->
