---
title: "Chuyên đề AI System Design: từ kiến trúc production-grade đến bảo mật, observability và voice Agent"
description: "Lộ trình học AI System Design và phỏng vấn về kiến trúc ứng dụng AI production-grade, bảo mật Agent, model gateway, OpenTelemetry Trace, AI observability, kiểm soát chi phí và voice Agent thời gian thực."
category: AI
tag:
  - AI System Design
  - LLM
  - Phát triển ứng dụng AI
  - AI observability
sidebar: false
---

<!-- @include: @small-advertisement.snippet.md -->

Prompt Demo chạy được chỉ cho thấy model đã đưa ra một câu trả lời dùng được trong một mẫu nào đó. Khi đưa vào production, vẫn cần giải quyết các vấn đề engineering: route model thế nào, fallback ra sao khi thất bại, phân bổ chi phí Token thế nào, khôi phục một request Agent ra sao, ai cấp quyền và audit các Tool nhạy cảm.

**Chuyên đề AI System Design** này dành cho các developer đã làm Demo và chuẩn bị đưa năng lực AI vào nghiệp vụ thực tế. Nội dung triển khai từ góc nhìn của backend system: phân tầng kiến trúc, model gateway, RAG, Memory, Tool calling, observability, đánh giá, bảo mật và pipeline voice Agent thời gian thực.

## Dành cho ai

- Developer đã làm Prompt hoặc RAG Demo và muốn biết trước khi đưa lên production còn thiếu những khâu engineering nào.
- Engineer cần thiết kế model gateway, routing nhiều model, fallback, rate limiting, cache và kiểm soát chi phí trong project.
- Bạn chuẩn bị cho câu hỏi phỏng vấn về AI System Design, model gateway, AI observability và voice Agent thời gian thực.

## Trọng tâm học

- Ứng dụng AI production-grade phải vận hành bền vững: chất lượng output, latency, fallback khi thất bại, chi phí và audit đều phải giải thích và review lại được.
- Model gateway tách model service khỏi bên gọi nghiệp vụ, thống nhất xử lý routing nhiều model, fallback, rate limiting, cache, ngân sách Token, phân bổ chi phí, observability, audit và security policy.
- Prompt management, RAG, Memory, Tool calling, async task, evaluation và observability cần được thiết kế trong cùng một pipeline; khó áp dụng một template chung, thường phải cân nhắc theo risk nghiệp vụ, lưu lượng gọi và ràng buộc chi phí.
- AI observability phải có thể đi sâu từ API request đến model, retrieval, Tool và Agent, đồng thời xử lý context propagation giữa các thread, sampling, dữ liệu nhạy cảm và việc đưa Badcase production trở lại.
- Ngoài text inference, voice Agent thời gian thực còn phải xử lý VAD, ASR, LLM, TTS, streaming playback, interruption và lựa chọn hybrid edge-cloud, nên nhạy cảm hơn với end-to-end latency.

## Thứ tự đọc đề xuất

1. [Thiết kế hệ thống ứng dụng AI](./ai-application-architecture.md): trước tiên xem cần bổ sung những năng lực backend nào khi đưa Prompt Demo vào production pipeline.
2. [Thực chiến bảo mật LLM/Agent](./llm-security.md): hiểu Prompt Injection trực tiếp và gián tiếp, resource authorization, approval, MCP authorization, isolation và security regression qua một lần Tool calling.
3. [Giải thích chi tiết LLM Gateway](./llm-gateway.md): tiếp tục xem cách quản trị model calling, routing nhiều model, fallback, rate limiting và phân bổ chi phí được xử lý ở gateway layer ra sao.
4. [AI observability và Trace](./ai-observability.md): khôi phục quá trình thực thi của model, RAG, Tool và Agent từ một câu trả lời lỗi, sau đó tìm hiểu context propagation, sampling và evaluation feedback.
5. [Giải thích chi tiết công nghệ voice AI](./ai-voice.md): cuối cùng tìm hiểu voice Agent, từ VAD, ASR đến LLM, TTS, playback và interruption.

## Bài viết cốt lõi

- [Thiết kế hệ thống ứng dụng AI](./ai-application-architecture.md): đi từ Prompt management đến model gateway, RAG, Memory, Tool calling, async task, observability, evaluation và security compliance; phù hợp làm trục chính để học System Design.
- [Thực chiến bảo mật LLM/Agent](./llm-security.md): kết nối Prompt Injection gián tiếp, Tool privilege escalation, MCP authorization, dữ liệu nhạy cảm, code isolation, supply chain và security evaluation qua một refund Agent hậu mãi.
- [Giải thích chi tiết LLM Gateway](./llm-gateway.md): làm rõ trách nhiệm của LLM Gateway trong model routing, fallback, rate limit quota, ngân sách Token, phân bổ chi phí, audit observability và chiến lược cache.
- [AI observability và Trace](./ai-observability.md): giải thích rõ Session, Run, Trace, Span và Attempt, đồng thời bao quát Agent/RAG/Tool Span, context propagation, sampling, dữ liệu nhạy cảm và tích hợp Spring AI.
- [Giải thích chi tiết công nghệ voice AI](./ai-voice.md): triển khai theo pipeline voice system, bao quát VAD, ASR, LLM, TTS, streaming playback, interruption và lựa chọn hybrid edge-cloud.

## Câu hỏi thường gặp

- Trước khi đưa Prompt Demo vào production còn thiếu những năng lực engineering nào?
- Khi Agent đọc tài liệu bên ngoài và gọi Tool ghi dữ liệu, làm thế nào để ngăn injection, privilege escalation và data exfiltration?
- Vì sao ứng dụng AI thường gom model calling về gateway layer?
- Routing nhiều model, fallback, rate limiting và cache lần lượt giải quyết vấn đề gì?
- Một request Agent nên được tách thành các Span thế nào, và làm sao duy trì Trace khi đi qua thread hoặc service khác?
- Làm sao đưa failed Trace trên production trở lại evaluation set và regression check?
- Vì sao latency, interruption và lựa chọn edge-cloud của voice Agent thời gian thực khó xử lý hơn?

## Tổng kết

Điểm khó của AI system production-grade không nằm ở việc chạy thành công một lần model calling, mà ở việc đưa quality, latency, cost, permission và fault handling vào cùng một pipeline có thể trace. Có thể bắt đầu từ application architecture để hiểu cách phân tầng các năng lực này, sau đó dùng model gateway để thống nhất việc quản trị calling, dùng Trace để khôi phục quá trình thực thi của RAG, Tool và Agent, cuối cùng kết hợp voice Agent để hiểu streaming, interruption và trade-off edge-cloud trong bối cảnh thời gian thực. Giải pháp cụ thể vẫn phải lựa chọn theo risk nghiệp vụ, quy mô gọi và yêu cầu compliance.

## Chuyên đề liên quan

- [Hệ thống kiến thức về phát triển ứng dụng AI](../)
- [Chuyên đề nền tảng LLM](../llm-basis/)
- [Chuyên đề AI Agent](../agent/)
- [Chuyên đề RAG](../rag/)
- [Chuyên đề câu hỏi phỏng vấn phát triển ứng dụng AI](../interview-questions/)

<!-- @include: @article-footer.snippet.md -->
