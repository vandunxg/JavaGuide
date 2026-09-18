---
title: Tổng hợp câu hỏi phỏng vấn về thiết kế hệ thống AI
description: Tổng hợp có hệ thống các câu hỏi phỏng vấn thường gặp về thiết kế hệ thống ứng dụng AI, bao quát các chủ đề cốt lõi như kiến trúc ứng dụng AI production-grade, model gateway, quản lý Prompt, RAG, Memory, Tool Calling, observability, đánh giá, bảo mật và tuân thủ, real-time voice Agent, kèm bài viết tham khảo tương ứng.
category: AI
tag:
  - Thiết kế hệ thống AI
  - Phỏng vấn AI
  - Ứng dụng LLM
head:
  - - meta
    - name: keywords
      content: câu hỏi phỏng vấn thiết kế hệ thống AI,câu hỏi phỏng vấn kiến trúc ứng dụng AI,thiết kế hệ thống ứng dụng LLM,câu hỏi phỏng vấn LLM Gateway,câu hỏi phỏng vấn observability AI,câu hỏi phỏng vấn đánh giá AI,câu hỏi phỏng vấn voice Agent,câu hỏi phỏng vấn bảo mật AI
---

Câu hỏi thiết kế hệ thống AI thường bắt đầu từ một kịch bản cụ thể, chẳng hạn như knowledge base doanh nghiệp, intelligent customer service, nền tảng Agent hoặc trợ lý voice theo thời gian thực. Khi vấn đề được mở rộng, interviewer sẽ tiếp tục hỏi cách đưa model invocation, context, retrieval, tool, security, cost và evaluation vào cùng một production pipeline.

Các câu hỏi được nhóm theo nội dung của chuyên đề thiết kế hệ thống AI trên JavaGuide. Mỗi nhóm có bài viết chi tiết đi kèm; tại đây chỉ tổng hợp các câu hỏi thường gặp, còn solution và implementation detail cụ thể nằm trong bài viết gốc tương ứng.

## Kiến trúc ứng dụng AI production-grade

Nội dung liên quan: [《Thiết kế hệ thống ứng dụng AI: Từ Prompt Demo đến kiến trúc production-grade》](../system-design/ai-application-architecture.md)

Câu hỏi kiến trúc sẽ bắt đầu từ toàn bộ request flow, sau đó kiểm tra trách nhiệm của từng module, cũng như cách lựa chọn giữa synchronous, streaming và asynchronous task. Prompt, RAG, Memory và Tool cũng cần được thảo luận trong đúng khâu mà chúng phụ trách.

Câu hỏi phỏng vấn thường gặp:

- Khoảng cách engineering giữa Prompt Demo và production system là gì?
- Làm thế nào để thiết kế kiến trúc tổng thể cho một ứng dụng AI production-grade?
- Một AI request từ lúc tiếp nhận đến khi trả về kết quả sẽ đi qua những module nào?
- Entry layer, orchestration layer, Prompt/Context, RAG/Memory/Tool, model gateway và evaluation/observability lần lượt phụ trách gì?
- Synchronous response, streaming response và asynchronous task lần lượt phù hợp với scenario nào?
- Vì sao Prompt cần được version management? Template, variable và model version nên liên kết với nhau thế nào?
- RAG, Memory và Tool lần lượt quản lý thông tin gì? Vì sao cần governance tách biệt?
- Làm thế nào để lưu intermediate state của long-running task? Khôi phục thế nào sau khi service restart?
- Để hỗ trợ request replay, tối thiểu cần ghi lại những data nào cho một request?

## Model gateway và governance invocation

Nội dung liên quan: [《Giải thích chi tiết về LLM Gateway: tích hợp thống nhất, model routing, rate limiting, quota và cost governance》](../system-design/llm-gateway.md), [《Thực tiễn engineering khi gọi LLM API: streaming output, retry, rate limiting và structured response》](../llm-basis/llm-api-engineering.md)

Câu hỏi về model gateway chủ yếu kiểm tra việc tích hợp nhiều provider, model routing và fault handling. Rate limiting không chỉ liên quan đến số request mà còn bao gồm Token, concurrency, tenant budget và upstream quota.

Câu hỏi phỏng vấn thường gặp:

- Vì sao production environment cần LLM Gateway? Việc business service gọi trực tiếp model API có những vấn đề gì?
- LLM Gateway khác LLM Router ở điểm nào?
- Model gateway thường phải đảm nhận những capability nào?
- Làm thế nào để thống nhất request parameter, response format và error code của nhiều model provider?
- Model routing có thể tham khảo những information nào? Làm thế nào để tránh phân request cho model không phù hợp?
- Vì sao rate limiting LLM cần đồng thời xét RPM, TPM, concurrency và tenant budget?
- Những model invocation error nào phù hợp để retry? Error nào nên fail ngay?
- Thiết kế model fallback thế nào? Task nào không được tự động downgrade?
- Làm thế nào để quy trách nhiệm Token cost cho tenant, user, feature, model và Prompt version?
- Model gateway sẽ tăng bao nhiêu latency? Những processing nào phù hợp để đặt trong gateway?
- Semantic cache phù hợp với request nào? Xử lý data freshness và permission isolation thế nào?
- Nếu được giao thiết kế một production-grade LLM Gateway, bạn sẽ chia module thế nào?

## Security, permission và audit

Nội dung liên quan: [《Thực chiến bảo mật LLM/Agent: từ Prompt Injection, tool privilege escalation đến sandbox isolation》](../system-design/llm-security.md), [《Thiết kế hệ thống ứng dụng AI: Từ Prompt Demo đến kiến trúc production-grade》](../system-design/ai-application-architecture.md), [《Structured output của LLM: từ JSON contract đến triển khai Function Calling》](../llm-basis/structured-output-function-calling.md)

Model có thể sinh tool calling intent và parameter, nhưng business operation thực tế vẫn do backend thực thi. Nhóm câu hỏi này sẽ tiếp tục hỏi identity, resource, parameter, operation risk và audit record nên được validate ở đâu.

Câu hỏi phỏng vấn thường gặp:

- Security boundary của Tool Calling nằm ở đâu?
- Vì sao tool description và Prompt không thể thay thế backend permission validation?
- Vì sao high-risk tool calling cần second confirmation?
- Write operation nên xử lý idempotency, timeout và vấn đề result uncertainty thế nào?
- Làm thế nào để phòng chống Prompt injection attack ở system design layer?
- Làm thế nào để RAG retrieval tránh retrieve content mà current user không có quyền xem?
- PII masking nên đặt ở input, log, model invocation hay output stage?
- Tool invocation audit log nên ghi lại những field nào?
- Sau khi structured parameter do model sinh ra đã qua Schema validation, vì sao vẫn không thể execute trực tiếp?

## Observability, evaluation và release

Nội dung liên quan: [《Hệ thống đánh giá ứng dụng AI: từ xây dựng Golden Set đến vòng lặp closed-loop của online canary》](../llm-basis/llm-evaluation.md), [《Thiết kế hệ thống ứng dụng AI: Từ Prompt Demo đến kiến trúc production-grade》](../system-design/ai-application-architecture.md)

Release check của ứng dụng AI không chỉ kiểm tra interface có thành công hay không mà còn phải bao quát answer quality, retrieval result, tool trace và structured output. Khi model, Prompt, retrieval config và code thay đổi, hệ thống cũng phải có khả năng compare và rollback.

Câu hỏi phỏng vấn thường gặp:

- AI application observability metric nên bao gồm những nội dung nào?
- Vì sao không có evaluation set thì rất khó phán đoán một thay đổi có hiệu quả hay không?
- Golden Set làm thế nào để bao phủ normal path, edge case, adversarial sample và high-risk failure?
- Offline evaluation, Trace replay và online canary lần lượt giải quyết vấn đề gì?
- Vì sao RAG, Agent và structured output không thể dùng chung một bộ evaluation metric?
- LLM-as-Judge có những bias nào? Làm thế nào để calibration bằng human sampling và rule validation?
- Làm thế nào để kiểm soát cost và runtime của AI evaluation trong CI?
- Vì sao evaluation record cần gắn với model, Prompt, retrieval config và code version?
- Khi online quality giảm, làm thế nào để phân biệt vấn đề do model, Prompt, retrieval, tool hay data distribution?
- Làm thế nào để thiết kế canary, rollback và failure sample feedback loop cho ứng dụng AI?

## Real-time voice Agent

Nội dung liên quan: [《Giải thích chi tiết công nghệ AI voice: từ ASR, TTS đến triển khai engineering real-time voice Agent》](../system-design/ai-voice.md)

Real-time voice system kết nối audio capture, VAD, ASR, LLM, tool calling, TTS và playback thành một pipeline. Các câu hỏi liên quan chủ yếu tập trung vào end-to-end latency, interruption handling, state management và lựa chọn giữa edge và cloud.

Câu hỏi phỏng vấn thường gặp:

- Làm thế nào để thiết kế một real-time voice Agent?
- ASR, LLM, TTS và VAD lần lượt phụ trách gì trong voice system?
- End-to-end latency của real-time voice Agent chủ yếu đến từ những khâu nào?
- Khi user interrupt, hệ thống làm thế nào để cancel playback, stop generation và update context?
- Làm thế nào để quản lý các state như `listening`, `thinking`, `speaking`, `interrupted`?
- Cascade ASR + LLM + TTS và native Speech-to-Speech model có những ưu, nhược điểm gì?
- Nên lựa chọn cloud API, local model hay hybrid edge-cloud solution thế nào?
- Audio preprocessing ở browser sẽ ảnh hưởng đến những metric nào?
- Observability data của voice Agent nên bao gồm những nội dung nào?

## Câu hỏi thiết kế tổng hợp

- Làm thế nào để thiết kế một multi-Agent system hỗ trợ task decomposition, parallel execution, state sharing, conflict handling và failure recovery? Tham khảo: [《Thiết kế hệ thống multi-Agent collaboration: task decomposition, state sharing, conflict handling và failure recovery》](../agent/multi-agent.md)
- Làm thế nào để thiết kế một enterprise RAG system có permission control, citation traceability và khả năng cập nhật knowledge base?
- Làm thế nào để thiết kế một Agent platform hỗ trợ long-running task, tool calling, interruption recovery và human takeover?
- Khi traffic của intelligent customer service đột ngột tăng, đồng thời model provider bắt đầu rate limiting, hệ thống nên queue, downgrade và bảo vệ core request thế nào?
- Sau khi model hoặc Prompt upgrade, structured output success rate và answer quality giảm, làm thế nào để xác định vấn đề và rollback?
- Khi Agent có thể query order và initiate refund, nên thiết kế permission, parameter validation, second confirmation, idempotency và audit thế nào?
- Làm thế nào để thiết kế một voice customer service system hỗ trợ real-time interruption, low latency và fault downgrade?
