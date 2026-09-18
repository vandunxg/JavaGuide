---
title: Các dự án AI open source chất lượng cho Java
description: Đề xuất các dự án AI open source chất lượng cho Java, bao gồm các framework và công cụ AI như Spring AI, LangChain4j, AgentScope, LangGraph4j, DJL.
category: Dự án open source
icon: "mdi:robot-outline"
---

Nếu bạn chủ yếu sử dụng Spring Boot, điều cần cân nhắc hiện nay không còn là “Java có thể phát triển ứng dụng AI hay không”, mà là dự án cần tích hợp model, RAG, điều phối Agent, hay chạy model trực tiếp trong JVM.

Các dự án này không hoàn toàn giải quyết cùng một vấn đề. Spring AI, LangChain4j thiên về tích hợp model và ứng dụng; AgentScope, LangGraph4j và Embabel tập trung hơn vào việc chạy và điều phối Agent; còn DJL dùng cho inference và training model. Trước khi chọn, hãy phân biệt rõ các tầng; cách này hiệu quả hơn việc chỉ so sánh số lượng tính năng.

## Framework nền tảng

### Spring AI

[Spring AI](https://github.com/spring-projects/spring-ai) là framework phát triển ứng dụng AI chính thức của Spring, nhằm kết nối model, vector database và các khả năng như tool calling bằng những abstraction theo phong cách Spring.

Với các dự án Spring Boot hiện có, cách tích hợp tương đối tự nhiên, chủ yếu cung cấp các abstraction sau:

- **Model communication (ChatClient)**: Sử dụng interface thống nhất để tương tác với các model như OpenAI, Ollama và Google Gemini.
- **Prompt**: Quản lý có cấu trúc các prompt gửi đến model.
- **Retrieval-Augmented Generation (RAG)**: Kết nối knowledge base bên ngoài với model thông qua các abstraction như `VectorStore`.
- **Tool calling (Function Calling)**: Cho phép model gọi các method được định nghĩa trong ứng dụng Java.
- **Memory (ChatMemory)**: Quản lý lịch sử context của các cuộc hội thoại nhiều lượt.

Tài liệu chính thức: <https://spring.io/projects/spring-ai#learn>.

### Spring AI Alibaba

[Spring AI Alibaba](https://github.com/alibaba/spring-ai-alibaba) tích hợp vào hệ sinh thái Spring AI, là một dự án được thiết kế riêng cho hệ thống multi-agent và điều phối workflow. Về kiến trúc, dự án gồm ba tầng sau:

![Kiến trúc Spring AI Alibaba](https://oss.javaguide.cn/github/javaguide/open-source-project/ai/springai-alibaba-architecture-new.png)

- **Agent Framework**: Framework phát triển Agent lấy ý tưởng thiết kế ReactAgent làm cốt lõi, xây dựng Agent có khả năng tự động context engineering và tương tác người-máy.
- **Graph**: Framework điều phối workflow và phối hợp multi-agent ở cấp thấp, là runtime nền tảng bên dưới của Agent Framework, hỗ trợ thực hiện việc điều phối ứng dụng phức tạp.
- **Augmented LLM**: Dựa trên abstraction nền tảng của Spring AI, cung cấp hỗ trợ cơ bản như model, tool, component đa phương thức (MCP) và vector storage.

Bên ngoài ba tầng này, dự án còn cung cấp hai component thiên về nền tảng:

- **Admin**: Nền tảng Agent tích hợp, hỗ trợ phát triển trực quan, observability, evaluation, quản lý MCP, thậm chí tích hợp với các nền tảng low-code như Dify và hỗ trợ migration DSL.
- **A2A (Agent-to-Agent)**: Hỗ trợ giao tiếp giữa các Agent và có thể tích hợp với Nacos để điều phối phân tán.

Tài liệu chính thức: <https://java2ai.com/>.

### LangChain4j

[LangChain4j](https://github.com/langchain4j/langchain4j) là framework ứng dụng LLM cho Java do cộng đồng duy trì, cung cấp interface thống nhất cho model, Embedding, vector storage, tool calling và RAG.

Framework này phù hợp với các dự án cần nhanh chóng chuyển đổi giữa nhiều model, vector database, hoặc muốn giữ framework tương đối độc lập. Khi dùng cho hệ thống lớn, vẫn cần bổ sung cấu trúc engineering, governance và observability dựa trên infrastructure hiện có.

Tài liệu chính thức: <https://docs.langchain4j.dev/>.

### AgentScope

[AgentScope](https://github.com/agentscope-ai/agentscope-java) là framework multi-agent, cung cấp các khả năng như suy luận ReAct, tool calling, quản lý memory và phối hợp multi-agent. Dự án đồng thời cung cấp phiên bản Python và Java.

Trọng tâm của nó khác với Spring AI Alibaba:

- **AgentScope Java**: Được thiết kế nguyên bản cho **paradigm Agentic**. Cốt lõi là “Agent”, nhấn mạnh tính tự chủ, vòng lặp suy luận (ReAct), cùng quá trình tương tác và phối hợp phức tạp giữa các multi-agent.
- **Spring AI Alibaba**: Tập trung hơn vào việc điều phối **Workflow**. Dựa trên hệ sinh thái Spring AI, framework này giỏi đưa năng lực AI vào business flow được định nghĩa trước dưới dạng tool.

Tài liệu chính thức: <https://java.agentscope.io/zh/intro.html>.

### Các framework và công cụ khác

- [Solon-AI](https://github.com/opensolon/solon-ai): Framework phát triển ứng dụng Java AI, hỗ trợ LLM, RAG, MCP và Agent, tương thích Java 8 đến Java 25, có thể kết hợp với các framework như Spring Boot, JFinal, Vert.x và Quarkus.
- [Agent-Flex](https://github.com/agents-flex/agents-flex): Framework phát triển ứng dụng Java LLM nhẹ, cung cấp các khả năng như tích hợp model, prompt, tool, memory, Embedding và vector storage.
- [Smile](https://github.com/haifengl/smile): Thư viện machine learning dựa trên Java và Scala.
- [LangGraph4j](https://github.com/langgraph4j/langgraph4j): Thư viện Java dùng để xây dựng ứng dụng stateful, multi-agent, có thể tích hợp với LangChain4j và Spring AI. Framework thiên về execution graph và quản lý state của Agent hơn, không phải một tầng tích hợp model mới.
- [Embabel Agent](https://github.com/embabel/embabel-agent): Framework Agent hướng đến JVM, kết hợp tương tác LLM, code Java/Kotlin và domain model thành flow Agent, đồng thời lập kế hoạch động cho execution path theo mục tiêu.
- [Deep Java Library](https://github.com/deepjavalibrary/djl): Framework deep learning cho Java, tách rời khỏi engine bên dưới, có thể load, training và deploy model trong ứng dụng Java. Framework giải quyết vấn đề inference và training model, không phải framework RAG hay điều phối Agent.

### So sánh

| **Tên framework**     | **Đặc điểm cốt lõi**                                                                                                                              | **Trường hợp sử dụng**                                                                                               |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| **Spring AI**         | Nền tảng chính thức của Spring: model/vector database/tool calling/memory/RAG/observability/structured output; nhấn mạnh tính portable và modular | AI hóa các ứng dụng enterprise Spring Boot hiện có                                                                   |
| **Spring AI Alibaba** | Hệ thống production-grade hướng đến Agentic/Workflow/Multi-agent: Agent Framework + Graph Runtime + Admin/Studio; hỗ trợ MCP/A2A/Nacos            | Điều phối multi-agent, workflow phức tạp, governance và migration theo nền tảng (bao gồm trực quan)                  |
| **LangChain4j**       | Thế mạnh cộng đồng: API thống nhất kết nối nhiều model/vector database; Agents/Tools/RAG; hỗ trợ MCP; có thể tích hợp Spring/Quarkus/Helidon      | Prototype nhanh, tính linh hoạt cao, chuyển đổi nhanh giữa nhiều model                                               |
| **Solon-AI**          | Tương thích Java 8~25; toàn bộ chuỗi LLM/RAG/MCP/Agent/Ai Flow; có thể nhúng vào nhiều framework                                                  | Hệ thống cũ/bối cảnh nhiều framework, yêu cầu tính tương thích và năng lực end-to-end                                |
| **Agent-Flex**        | Nhẹ: LLM/Prompt/Tool/MCP/Memory/Embedding/VectorStore/xử lý tài liệu; observability với OpenTelemetry                                             | Ứng dụng LLM cần tích hợp nhanh, giữ ít dependency                                                                   |
| **AgentScope Java**   | Agentic nguyên bản: ReAct + Tool + Memory + multi-agent; MCP+A2A (Nacos); reactive với Reactor + Serverless GraalVM                               | Agent tự chủ, multi-agent phân tán, các trường hợp yêu cầu cao về khả năng kiểm soát và performance trong production |

## Thực chiến

### Nền tảng phỏng vấn thông minh

[interview-guide](https://github.com/Snailclimb/interview-guide) dựa trên Spring Boot 4.0 + Java 21 + Spring AI + PostgreSQL + pgvector + RustFS + Redis, triển khai các tính năng cốt lõi như phân tích CV thông minh, phỏng vấn mô phỏng bằng AI và truy vấn RAG từ knowledge base. Dự án rất phù hợp làm dự án học tập và dự án trong CV, ngưỡng học thấp.

**Kiến trúc hệ thống như sau**:

> **Gợi ý**: Sơ đồ kiến trúc được vẽ bằng draw.io và export ở định dạng SVG, hiệu ứng hiển thị trong dark mode của GitHub sẽ có vấn đề.

![Sơ đồ kiến trúc hệ thống](https://oss.javaguide.cn/xingqiu/pratical-project/interview-guide/interview-guide-architecture-diagram.png)

### Hệ thống điều phối workflow AI

[PaiAgent](https://github.com/itwanger/PaiAgent) là một nền tảng điều phối workflow AI trực quan, có thể kết hợp model và các node xử lý bằng thao tác kéo thả để định nghĩa flow thực thi phối hợp nhiều model.

**Kiến trúc hệ thống như sau**:

![](https://oss.javaguide.cn/github/javaguide/open-source-project/ai/paiagent-architecture-diagram.jpg)
