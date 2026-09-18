---
title: "Hệ thống kiến thức phát triển ứng dụng AI: LLM, Agent, RAG, MCP, Prompt Engineering và thiết kế hệ thống"
description: "Lộ trình học và phỏng vấn phát triển ứng dụng AI, hệ thống hóa các nội dung về gọi LLM, Agent, RAG, Skills, MCP, Prompt Engineering, vector database, evaluation và thiết kế hệ thống dành cho backend developer."
category: AI
tag:
  - AI
  - LLM
  - AI application development
  - phỏng vấn backend
icon: mdi:robot-outline
sitemap:
  changefreq: weekly
  priority: 1
head:
  - - meta
    - name: keywords
      content: phát triển ứng dụng AI,phỏng vấn phát triển ứng dụng AI,phỏng vấn kỹ sư AI,LLM,phỏng vấn LLM,Agent,phỏng vấn Agent,RAG,phỏng vấn RAG,MCP,Prompt Engineering,vector database,thiết kế hệ thống AI,phỏng vấn AI coding
  - - meta
    - property: og:title
      content: "Hệ thống kiến thức phát triển ứng dụng AI: LLM, Agent, RAG, MCP, Prompt Engineering và thiết kế hệ thống"
  - - meta
    - property: og:description
      content: "Hệ thống hóa các nội dung từ gọi LLM, Agent, RAG, MCP, Prompt Engineering đến evaluation và thiết kế hệ thống, tập trung vào những kiến thức then chốt backend developer cần bổ sung khi chuyển sang phát triển ứng dụng AI."
---

<!-- @include: @small-advertisement.snippet.md -->

Xây dựng ứng dụng AI không phải chỉ nhét Prompt vào API là xong. Khi đi vào dự án thực tế, bạn sẽ lập tức gặp các vấn đề như độ dài context, structured output, RAG retrieval, quyền của tool, evaluation regression, cost và stability.

Những vấn đề này không thể giải quyết riêng lẻ. Cần hiểu liền mạch nền tảng LLM, Agent, RAG, tool calling và system design. Chỉ biết gọi API thì sẽ bị chặn ở bước architecture review; chỉ quen với các paper về RAG thì khi bảo trì knowledge base vẫn không biết xử lý incremental update và version deduplication.

Nếu thời gian có hạn, hãy đọc trước [Hướng dẫn phỏng vấn phát triển ứng dụng AI](./interview-questions/ai-interview-guide.md) để xem qua các vấn đề dễ bị hỏi sâu nhất trong LLM, Agent, RAG, Skills, MCP và AI system design; nếu chưa xác định được thứ tự học hoặc đang chuyển từ backend development sang phát triển ứng dụng AI, bạn có thể đọc trước [Lộ trình học phát triển ứng dụng AI và Agent dành cho developer Java/Go (bản mới nhất 2026)](../roadmap/java-to-ai-roadmap.md) và [Gợi ý học chuyển hướng sang AI Agent dành cho backend developer (bản mới nhất 2026)](../roadmap/backend-to-ai-agent-roadmap.md); nếu muốn học vững hơn, hãy tiếp tục theo thứ tự đọc bên dưới.

Nội dung chuyên đề được tổ chức theo nền tảng LLM, Agent, RAG và system design; các bài viết được liên kết với nhau qua thứ tự đọc và các liên kết liên quan:

![Tổng quan nội dung AIGuide với nhiều hình minh họa](https://oss.javaguide.cn/github/aiguide/aiguide-overview.png)

Nội dung chuyên mục này cũng được tổng hợp trong project mã nguồn mở AIGuide:

- **Địa chỉ project**: [https://github.com/Snailclimb/AIGuide](https://github.com/Snailclimb/AIGuide)
- **Đọc online**: [https://javaguide.cn/ai-coding/](https://javaguide.cn/ai-coding/)

Bài viết sẽ được cập nhật liên tục theo thay đổi của API, framework và năng lực model. Khi đề cập đến version, price và product capability, hãy đồng thời đối chiếu tài liệu chính thức tương ứng.

## Dành cho ai

- Kỹ sư đang chuyển từ backend development sang phát triển ứng dụng AI và muốn bổ sung tuyến kiến thức chính về LLM, Agent, RAG và system design.
- Người chuẩn bị phỏng vấn các vị trí AI engineer, AI application development hoặc backend chuyển sang AI.
- Developer từng làm Prompt Demo nhưng chưa thật sự quen với model invocation flow, structured output, tối ưu RAG retrieval và evaluation loop.
- Người muốn hiểu các khái niệm như MCP, Function Calling, Tool Calling, vector database và model gateway trong project thực tế.
- Thành viên team đã tích hợp LLM vào project nhưng bắt đầu gặp các vấn đề về stability, cost, security governance và quality regression.

## Một số điểm dễ mắc lỗi

Không nên chỉ xem LLM như một black-box API. Token bị truncate, output thay đổi thất thường khi sampling parameter thay đổi, hoặc kết quả vẫn sai format dù đã yêu cầu JSON, đều là những vấn đề khó giải quyết triệt để chỉ bằng Prompt. Thêm câu “hãy output đúng theo JSON” vào prompt chỉ là constraint ở tầng đầu tiên; khi đưa vào production, vẫn cần validation format, retry, fallback và exception handling trong invocation flow.

Agent cũng không phải chỉ cần tự động gọi tool là xong. Điều thực sự khó là Memory và Context Engineering. Nếu không quản lý context tốt, sau vài vòng chạy Agent rất dễ đi lệch chủ đề; những gì đã nói, task đang làm đến đâu và kết quả tool nào còn dùng được đều có thể bị rối. Điều này càng rõ trong task dài: đôi khi Agent không phải không làm được, mà là sau vài vòng lặp lại tự mắc kẹt và chạy mãi đến khi token gần cạn mới dừng.

RAG trả lời không đúng câu hỏi nhiều khi cũng không nên vội trách model. Phần lớn vấn đề nằm ở retrieval stage: Chunk quá thô, Query chưa được rewrite, keyword search và vector search chưa kết hợp, hoặc rerank chưa tốt. Khi đó, kiểm tra từng bước trong retrieval flow thường hữu ích hơn việc đổi ngay sang một model đắt hơn.

MCP, Function Calling và Tool Calling giải quyết vấn đề kết nối tool vào hệ thống. Sau khi protocol được thống nhất, việc kết nối tool quả thực thuận tiện hơn, nhưng trong production environment, vấn đề lại nằm ở phía sau: ai được gọi tool này, được thao tác trên những data nào, audit invocation record ra sao và rollback thế nào khi thất bại. Nếu không thiết kế tốt các phần này, protocol có standard đến đâu cũng chưa đủ.

Sau khi ứng dụng AI được đưa lên production, các vấn đề về stability, observability, cost control và quality regression sẽ lần lượt xuất hiện. Giai đoạn Demo thường chưa cảm nhận được vì invocation volume nhỏ và scenario cũng đơn giản. Khi thật sự tiếp nhận business traffic, team lần đầu xây dựng production-grade AI application gần như đều phải trả giá một lần cho những vấn đề này.

## Thứ tự đọc đề xuất

1. [Tổng quan các khái niệm cốt lõi về AI](./ai-core-concepts.md): Trước hết đặt các khái niệm LLM, Token, Agent, RAG, MCP, Skills và ReAct vào cùng một flow.
2. [Hướng dẫn phỏng vấn phát triển ứng dụng AI](./interview-questions/ai-interview-guide.md): Xây dựng danh sách câu hỏi thường gặp và biết những điểm thường bị hỏi sâu nhất trong phỏng vấn và project review.
3. [Cơ chế hoạt động của LLM](./llm-basis/llm-operation-mechanism.md), [Thực hành engineering khi gọi API của LLM](./llm-basis/llm-api-engineering.md): Hiểu invocation flow, context và structured response của model.
4. [Khái niệm cốt lõi về AI Agent](./agent/agent-basis.md), [Prompt Engineering cho LLM](./agent/prompt-engineering.md), [Context Engineering](./agent/context-engineering.md): Xây dựng nhận thức nền tảng về Agent, Prompt và Context.
5. [Thiết kế hệ thống cộng tác multi-Agent](./agent/multi-agent.md): Tiếp tục học cách phân tách task, chia sẻ state, xử lý conflict và recovery khi thất bại.
6. [Khái niệm nền tảng về RAG](./rag/rag-basis.md), [Xử lý và chunk tài liệu RAG](./rag/rag-document-processing.md), [Tối ưu RAG retrieval](./rag/rag-optimization.md): Bổ sung tuyến kiến thức chính về hỏi đáp trên enterprise knowledge base.
7. [Thiết kế hệ thống ứng dụng AI](./system-design/ai-application-architecture.md), [Thực chiến security cho LLM/Agent](./system-design/llm-security.md), [Giải thích chi tiết LLM Gateway](./system-design/llm-gateway.md), [Hệ thống evaluation cho ứng dụng AI](./llm-basis/llm-evaluation.md): Đưa Demo vào backend system thực tế, bổ sung permission, security, gateway, evaluation và governance.

## Bài viết cốt lõi

### Lộ trình phỏng vấn và ôn tập

- [Lộ trình học phát triển ứng dụng AI và Agent dành cho developer Java/Go (bản mới nhất 2026)](../roadmap/java-to-ai-roadmap.md): Phân tách lộ trình học theo nền tảng LLM, LLM API, Prompt, RAG, Agent, engineering và project thực chiến.
- [Gợi ý học chuyển hướng sang AI Agent dành cho backend developer (bản mới nhất 2026)](../roadmap/backend-to-ai-agent-roadmap.md): Trước hết đánh giá có phù hợp để chuyển hướng không, sau đó xem nên chọn Java AI hay Python AI, có thể ứng tuyển vị trí nào và nên học như thế nào.
- [Tổng quan các khái niệm cốt lõi về AI](./ai-core-concepts.md): Liên kết các khái niệm cốt lõi LLM, Token, MCP, Skills, ReAct, Embedding và GraphRAG theo ba tuyến nền tảng LLM, Agent và RAG.
- [Chuyên đề câu hỏi phỏng vấn phát triển ứng dụng AI](./interview-questions/): Tổ chức lộ trình ôn tập theo nền tảng LLM, AI Agent, RAG và AI system design.
- [Hướng dẫn phỏng vấn phát triển ứng dụng AI](./interview-questions/ai-interview-guide.md): Đặt các câu hỏi thường gặp khi phát triển ứng dụng AI vào một lộ trình ôn tập, phù hợp để đọc trước.
- [Tổng hợp câu hỏi phỏng vấn nền tảng LLM](./interview-questions/llm-interview-questions.md): Bao quát Token, context window, sampling parameter, API invocation, structured output và evaluation system.
- [Tổng hợp câu hỏi phỏng vấn AI Agent](./interview-questions/agent-interview-questions.md): Bao quát Agent Loop, Memory, Prompt, Context, MCP, Skills, Harness Engineering và workflow.
- [Trình bày project Agent trong phỏng vấn như thế nào?](./interview-questions/agent-project-interview-guide.md): Trình bày từ system architecture, technology selection, evidence về hiệu quả đến review Badcase, phù hợp để hệ thống hóa project thực tế thành câu trả lời phỏng vấn hoàn chỉnh.
- [Tổng hợp câu hỏi phỏng vấn RAG](./interview-questions/rag-interview-questions.md): Bao quát nền tảng RAG, vector database, document processing, retrieval optimization, GraphRAG, knowledge base update và evaluation.
- [Tổng hợp câu hỏi phỏng vấn thiết kế hệ thống AI](./interview-questions/ai-system-design-interview-questions.md): Bao quát production-grade AI application architecture, model gateway, observability, evaluation, security governance và real-time voice Agent.

### Nền tảng LLM

- [Chuyên đề nền tảng LLM](./llm-basis/): Từ model operation mechanism, API invocation, structured output đến AI application evaluation, trước hết làm rõ invocation flow.
- [Cơ chế hoạt động của LLM: Token, context window và sampling parameter ảnh hưởng đến output như thế nào](./llm-basis/llm-operation-mechanism.md): Chuyển các khái niệm Token, context window và Temperature thành những engineering parameter rõ ràng, có thể kiểm soát.
- [Thực hành engineering khi gọi API của LLM](./llm-basis/llm-api-engineering.md): Phân tích Prompt assembly, model gateway, streaming response, retry, rate limiting và structured response.
- [Giải thích chi tiết structured output của LLM](./llm-basis/structured-output-function-calling.md): Làm rõ JSON Schema, Function Calling, Tool Calling và flow bên trong của MCP.
- [Hệ thống evaluation cho ứng dụng AI](./llm-basis/llm-evaluation.md): Bao quát Golden Set, LLM-as-Judge, metric của RAG/Agent, Trace replay và production canary loop.

### AI Agent

- [Chuyên đề AI Agent](./agent/): Từ khái niệm nền tảng Agent, Memory, Prompt, Context đến MCP, Skills và Harness Engineering.
- [Khái niệm cốt lõi về AI Agent](./agent/agent-basis.md): Hiểu sự khác biệt giữa Agent với traditional programming và Workflow, cùng các khái niệm cốt lõi như Agent Loop và Tools registration.
- [Hệ thống memory của AI Agent](./agent/agent-memory.md): Tìm hiểu sâu về short-term memory, long-term memory, memory lifecycle và chiến lược tối ưu production-grade.
- [Thiết kế hệ thống cộng tác multi-Agent](./agent/multi-agent.md): Làm rõ khi nào cần multi-Agent, cũng như cách triển khai task decomposition, state sharing, conflict handling và failure recovery.
- [Prompt Engineering cho LLM](./agent/prompt-engineering.md): Nắm vững bốn yếu tố của Prompt, các kỹ thuật thường dùng và cách phòng chống Prompt Injection.
- [Context Engineering](./agent/context-engineering.md): Hiểu static rule orchestration, dynamic information mounting, Token budget degradation và context persistence.
- [Phân tích chi tiết MCP protocol](./agent/mcp.md): Hiểu layered architecture, core capability và cách triển khai MCP Server trong production.
- [Giải thích chi tiết Agent Skills](./agent/skills.md): Hiểu khác biệt bản chất giữa Skills với Prompt, MCP và Function Calling.
- [Harness Engineering: framework kiểm tra sáu tầng, quản lý context và thực hành engineering](./agent/harness-engineering.md): Phân tích architecture engineering của Model + Harness và thực hành team.
- [Workflow, Graph và Loop trong AI workflow](./agent/workflow-graph-loop.md): Hiểu node, edge, state, security boundary và cách triển khai AI workflow.
- [Loop Engineering là gì? Vì sao nói đây là bình mới rượu cũ?](./agent/loop-engineering.md): Giải thích trigger, context, validation, state và stopping condition của outer loop trong code Agent.

### RAG (Retrieval-Augmented Generation)

- [Chuyên đề RAG](./rag/): Xoay quanh hỏi đáp trên enterprise knowledge base, hệ thống hóa document processing, vector database, GraphRAG, retrieval optimization và knowledge base update.
- [Khái niệm nền tảng về RAG](./rag/rag-basis.md): Hiểu RAG là gì, vì sao cần RAG, ưu điểm cốt lõi và giới hạn của nó.
- [Xử lý và chunk tài liệu RAG](./rag/rag-document-processing.md): Bao quát document parsing, cleaning, structuring, chunking và xử lý nội dung multimodal.
- [Thuật toán vector index và vector database của RAG](./rag/rag-vector-store.md): Nắm vững các index algorithm như HNSW, IVFFLAT và cách chọn vector database.
- [Tối ưu RAG retrieval](./rag/rag-optimization.md): Bao quát Chunk strategy, Hybrid Search, Query Rewrite, Rerank và context compression.
- [GraphRAG](./rag/graphrag.md): Hiểu entity, relation, community detection, global retrieval và local retrieval.
- [Chiến lược cập nhật tài liệu trong RAG knowledge base](./rag/rag-knowledge-update.md): Nắm vững incremental update, version control, deduplication và full rebuild.

### Thiết kế hệ thống AI

- [Chuyên đề thiết kế hệ thống AI](./system-design/): Đưa Prompt Demo vào backend system thực tế, tập trung vào architecture, model gateway, voice flow, observability, evaluation và security governance.
- [Thiết kế hệ thống ứng dụng AI](./system-design/ai-application-architecture.md): Đưa Prompt Demo vào production flow, bao quát Prompt management, model gateway, RAG, Memory, Tool invocation, observability, evaluation và security compliance.
- [Thực chiến security cho LLM/Agent](./system-design/llm-security.md): Bao quát direct và indirect Prompt Injection, tool privilege escalation, MCP authorization, sensitive data, code sandbox, supply chain và security regression.
- [Giải thích chi tiết LLM Gateway](./system-design/llm-gateway.md): Hiểu multi-model routing, fallback, rate limit quota, cost attribution, observability audit và caching strategy của LLM Gateway.
- [Giải thích chi tiết công nghệ voice AI](./system-design/ai-voice.md): Phân tích VAD, ASR, LLM, TTS, streaming playback, interruption handling và lựa chọn hybrid giữa edge và cloud.

## Câu hỏi thường gặp

- Token, context window, Temperature và Top P của LLM lần lượt ảnh hưởng đến điều gì?
- Vì sao structured output không thể chỉ dựa vào Prompt? JSON Schema, Function Calling và server-side validation lần lượt giải quyết vấn đề gì?
- Agent khác Workflow ở điểm nào? Observation, planning, action và reflection phối hợp với nhau như thế nào trong Agent Loop?
- Prompt Engineering khác Context Engineering ở điểm nào?
- MCP giải quyết vấn đề gì? Mối quan hệ giữa MCP với Function Calling và Tool Calling là gì?
- Vì sao RAG trả lời không đúng câu hỏi? Nên kiểm tra từ retrieval, ranking, context compression hay generation stage?
- Nên chọn vector database như thế nào? Các index như HNSW và IVFFLAT phù hợp với scenario nào?
- Đánh giá ứng dụng AI như thế nào? Golden Set, LLM-as-Judge, production canary và Trace replay được liên kết với nhau ra sao?
- Vì sao production-grade AI application cần model gateway? Làm thế nào để thực hiện rate limiting, fallback, cost control và audit?

## Chuyên đề liên quan

- [Hướng dẫn thực chiến AI coding](../ai-coding/)
- [Thiết kế hệ thống](../system-design/)
- [Hệ thống kiến thức về high availability](../high-availability/)
- [Hệ thống kiến thức về high performance](../high-performance/)
- [Hệ thống kiến thức về distributed system](../distributed-system/)

<!-- @include: @article-footer.snippet.md -->
