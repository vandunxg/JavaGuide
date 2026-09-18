---
title: "Workflow, Graph và Loop trong AI Workflow: từ khái niệm đến triển khai"
description: Phân tích ba khái niệm Workflow, Graph và Loop trong AI Workflow, so sánh sự khác biệt giữa Workflow truyền thống và AI Workflow, đồng thời dùng Spring AI Alibaba Graph để minh họa cách triển khai State, conditional edge và vòng lặp.
category: AI Application Development
icon: "mdi:robot-outline"
head:
  - - meta
    - name: keywords
      content: AI Workflow,Graph,Loop,AI Workflow,Spring AI Alibaba,LangGraph,state machine,Agent,workflow engine
---

Các engine truyền thống như Camunda, Temporal cũng hỗ trợ event, branch, retry và compensation. Điểm phức tạp mới của AI Workflow là output của một số node được LLM tạo ra, còn “kết quả có đạt yêu cầu hay không” cũng có thể cần model hoặc scorer đánh giá trong runtime. Vì vậy, flow thường xuất hiện back edge: generate, evaluate, modify rồi quay lại evaluate.

Workflow mô tả cách hoàn thành task, Graph dùng Node, Edge và State để biểu đạt cấu trúc thực thi, còn Loop là cơ chế quay lui trên Graph. Dưới đây sẽ dùng ví dụ review bài viết để giải thích cách ba khái niệm phối hợp, đồng thời dùng Spring AI Alibaba Graph minh họa việc cập nhật State và conditional edge.

## Vì sao AI system cần workflow?

Một lượt hội thoại có thể trả lời câu hỏi, nhưng khó **bàn giao kết quả** một cách ổn định. Task thực tế trên production hiếm khi chỉ là “hỏi một câu, trả lời một câu” rồi kết thúc: tìm kiếm thông tin, gọi tool, xuất kết quả có cấu trúc, kiểm tra format, retry khi thất bại, thực hiện thêm một lượt khi chưa đạt yêu cầu, chỉ khi nối các bước này lại mới tạo thành việc bàn giao kết quả. Nhồi toàn bộ logic vào một Prompt quá dài sớm muộn cũng sẽ hỏng. Bạn cần một execution path **có thể phân nhánh, lặp và quan sát được**.

Business flow truyền thống thường định nghĩa trước các bước và quy tắc branch có thể có, nhưng node vẫn có thể tạo ra kết quả khác nhau do thao tác thủ công hoặc external API. Sau khi thêm LLM, nội dung được generate và việc đánh giá chất lượng lại thêm một lớp không chắc chắn. Điều này tạo ra ba vấn đề trực tiếp:

1. Bước tiếp theo không duy nhất, cần quyết định path động dựa trên kết quả hiện tại;
2. Khi kết quả chưa tốt, system cần tự động sửa thay vì thất bại ngay;
3. Phải ghi lại State trung gian, nếu không sẽ khó debug, trace và recovery.

Đó cũng là lý do AI system cần tư duy workflow.

Xét một ví dụ đơn giản: khi yêu cầu AI viết một bài viết, kết quả generate một lần thường chưa lý tưởng. Cách làm theo trực giác là copy kết quả thủ công, thêm yêu cầu mới rồi tiếp tục hỏi, nhưng cách này vừa không hiệu quả vừa nhanh chóng tiêu hao context. Nếu cấu trúc hóa quá trình này thành vòng lặp “**review → revise → review lại**”, đồng thời đặt stopping condition (như đạt tiêu chuẩn chất lượng hoặc chạm giới hạn iteration), tính ổn định sẽ tốt hơn rõ rệt.

Nói cho cùng, workflow là biến quá trình generate một lần thành một flow có hệ thống **có thể iteration, hội tụ và kiểm soát**.

## Workflow truyền thống và AI Workflow khác nhau thế nào?

![So sánh Workflow truyền thống và AI Workflow](https://oss.javaguide.cn/github/javaguide/ai/workflow/traditional-vs-ai-workflow.svg)

Hình trên cho thấy trực quan sự khác biệt giữa hai loại workflow: Workflow truyền thống thiên về quá trình orchestration “step cố định + branch rõ ràng”; AI Workflow phụ thuộc nhiều hơn vào State tại runtime để quyết định động bước tiếp theo, đồng thời biến “generate — evaluate — revise” thành một quá trình có thể hội tụ thông qua Loop.

### Đặc điểm của workflow truyền thống

Trước hết là định nghĩa cơ bản: **Workflow** là việc chia task thành một số step để hoàn thành một mục tiêu, đồng thời quy định cách các step phối hợp và tiến triển. Nó trả lời câu hỏi: “Làm thế nào để hoàn thành việc này?”

Workflow truyền thống cũng có thể orchestration task thủ công, external API và các hoạt động không deterministic khác, vì vậy “cùng input chắc chắn nhận được cùng kết quả node” không phải tiền đề của nó. Điểm khác phổ biến hơn là: flow truyền thống định nghĩa trước activity, branch có thể có và quy tắc compensation, rồi chọn path tại runtime dựa trên event và business data. BPMN 2.0, Camunda, Temporal, Apache Airflow đều không bị giới hạn ở thứ tự tuyến tính.

Điểm khác biệt cốt lõi giữa AI Workflow và workflow truyền thống là việc chọn path phụ thuộc vào đánh giá chất lượng của nội dung được generate tại runtime, đồng thời cùng một node có thể cần thực thi lặp lại do output không chắc chắn. Ví dụ trong các scenario truyền thống như flow approval, order flow, pipeline dữ liệu ETL, điều kiện branch là rõ ràng (amount > 10000 thì chuyển sang approval cấp cao); còn trong scenario AI, bản thân phán đoán “kết quả generate có đạt yêu cầu không” đã cần được đánh giá tại runtime, và kết luận đánh giá có thể khiến flow quay lại step trước để liên tục revise.

### Đặc điểm của AI Workflow

Trong scenario AI, cùng là từ “flow” nhưng ý nghĩa đã khác. So với tính tuần tự và deterministic mà workflow truyền thống nhấn mạnh, AI Workflow phải xử lý một execution environment đầy không chắc chắn. Điều chúng ta đối mặt không chỉ là “thực thi theo step”, mà còn bao gồm:

- Có đạt yêu cầu hay không phải được phán đoán tại **runtime**.
- Có cần retry tiếp hay không phải do **State hiện tại** quyết định.
- Sau khi một step thất bại, system không chỉ báo lỗi rồi kết thúc, mà còn phải cân nhắc có nên degrade, rollback hay đổi strategy.
- Dữ liệu truyền giữa các node không chỉ là parameter, mà còn gồm context, draft, score, error message, các lượt trước và **State** khác.

Vì vậy, AI Workflow và Workflow truyền thống đều có flow, nhưng điểm khác là loại trước nhấn mạnh dynamic decision và state-driven. Một khi muốn biểu đạt “bước tiếp theo không duy nhất” hoặc “chưa hài lòng thì làm thêm một lượt”, một linear list sẽ không đủ, và tự nhiên sẽ đi đến hai khái niệm Graph (structure) và Loop (quay lui).

## Graph và Loop là gì?

### Graph: structure của workflow

Tiếp tục case xuyên suốt: giả sử cần xây dựng một path “generate draft → quality review → nếu chưa đạt thì revise → quay lại review”. Mỗi step ở đây tương ứng với **Node** của graph, hướng đi giữa các step được biểu đạt bằng **Edge**, còn shared context được cả chain đọc và ghi là **State**.

Graph có ba element cơ bản nhất:

- **Node**: execution unit, chức năng chính là đọc State, thực thi logic và cập nhật State. Các node điển hình trong ví dụ review bài viết gồm “generate draft”, “quality review”, “revise theo feedback”, ngoài ra có thể mở rộng thêm retrieval, format validation, human approval và các node khác.
- **Edge**: abstraction của control flow, quyết định execution path giữa các node. Các loại edge thường gặp:
  - **Sequential edge**: node thực thi theo thứ tự cố định, không phụ thuộc vào conditional judgment
  - **Conditional edge**: chọn trong các path đã định nghĩa trước dựa trên State tại runtime, Spring AI Alibaba triển khai bằng `addConditionalEdges()`
  - **Dynamic routing**: node ứng viên được xác định động tại runtime, ví dụ API `Send` của LangGraph có thể quyết định động số lần gọi song song
  - **Loop edge**: node quay về chính nó hoặc node trước đó để thực thi lặp, dùng cho retry và iteration
  - **Termination edge**: flow kết thúc, không thực thi các node tiếp theo
  - **Parallel edge**: một node đồng thời phân phối đến nhiều node tiếp theo để thực thi song song

> Trong engineering thực tế, conditional edge và dynamic routing là một spectrum liên tục: candidate set của conditional edge được xác định khi design nhưng logic lựa chọn có thể phụ thuộc vào State tại runtime (như LLM score), còn candidate set của dynamic routing chỉ được xác định tại runtime (như API `Send` của LangGraph tạo động các parallel branch). Trong phần lớn scenario, conditional edge đã đủ dùng; dynamic routing phù hợp với các scenario như map-reduce, cần quyết định số lượng parallel branch tại runtime.

- **State**: biểu thị shared context liên tục được đọc và ghi trong quá trình thực thi flow, là “working memory” thực sự được truyền giữa các node. Cách triển khai phổ biến là **key-value data structure** (tương tự Java `Map<String, Object>`, Python `dict`, TypeScript `Record<string, any>`), dùng để truyền và sửa data giữa các node.

Cần lưu ý, design của State không chỉ liên quan đến “lưu gì”, mà còn liên quan đến “update thế nào”. Trong các workflow framework thực tế, các field khác nhau thường có update semantic khác nhau:

- **Replace**: value mới trực tiếp thay thế value cũ. Phù hợp với field single-value như classification result, current state. Trong Spring AI Alibaba tương ứng với `ReplaceStrategy`, trong LangGraph tương ứng với behavior mặc định không có reducer.
- **Append**: value mới được append vào list hiện có. Phù hợp với field dạng tích lũy như conversation history (`messages`). Trong Spring AI Alibaba tương ứng với `AppendStrategy`, trong LangGraph tương ứng với `Annotated[list, operator.add]`.
- **Custom Reducer**: quyết định logic merge thông qua function tùy chỉnh, ví dụ `add_messages` của LangGraph sẽ append hoặc update dựa trên message ID.

Khi nhiều node song song cùng ghi vào một field dùng semantic Replace, sẽ phát sinh vấn đề race (LangGraph sẽ throw error `INVALID_CONCURRENT_GRAPH_UPDATE`). Vì vậy, khi design State cần lên kế hoạch trước field nào có thể được ghi song song và chọn update strategy phù hợp cho chúng.

Các State field thường dùng trong project thực tế (có thể điều chỉnh theo yêu cầu business):

- `input`: user input, giữ lại trong toàn bộ flow
- `messages`: conversation history, dùng Append strategy
- `retrieval_result`: kết quả retrieval của RAG, State trung gian
- `tool_result`: kết quả gọi tool, State trung gian
- `llm_response`: output gốc của LLM, State trung gian
- `intermediate_steps`: record các step thực thi trung gian, giữ lại trong toàn bộ flow
- `next_step`: node chuyển control flow (Spring AI Alibaba dùng field này kết hợp với conditional edge để routing; LangGraph dùng trực tiếp return value của conditional edge function nên không cần field này)
- `output`: kết quả output cuối cùng

Nếu chỉ nhìn Node và Edge, ta sẽ có một “path graph có thể chạy”; thêm State thì graph này mới có thể ra quyết định tại runtime.

Cấu trúc Graph gần với hình thái thực tế của AI system hơn cấu trúc tuyến tính, vì control flow của nhiều AI application vốn đã là graph, chỉ là trước đây thường được viết tạm thành `if-else`, logic retry hoặc state machine rải rác trong các module khác nhau.

### Loop: quay lui trên Graph

Trong cùng case “review bài viết”: khi **review không đạt**, control flow không nên kết thúc, mà nên đi theo một edge quay về “revise” hoặc “generate lại”. Đây là ý nghĩa của Loop trong business. Về mặt kỹ thuật, nó biểu hiện thành **Back Edge** trên graph.

> Cần phân biệt Loop trong bài này với **Agent Loop** trong bài Agent Basics. Agent Loop là execution engine cấp cao nhất của Agent: toàn bộ Agent lặp lại “reasoning → action → observation” trong một vòng while cho đến khi task hoàn thành. Còn Loop trong bài này là control pattern bên trong Graph: một subset node cụ thể tạo thành vòng lặp iteration và revise thông qua back edge. Quan hệ giữa hai bên là: Agent Loop là outer loop, còn Graph Loop có thể được nest trong một node hoặc subgraph nào đó của nó.

![Tổng quan Loop: sơ đồ cơ chế vòng lặp](https://oss.javaguide.cn/github/javaguide/ai/workflow/loop-mechanism.svg)

Nhiều người lần đầu tiếp xúc với AI Workflow sẽ hiểu `Loop` là “chạy thêm vài lần”. Cách hiểu này không sai, nhưng chưa đủ chính xác. Nói chính xác hơn: **Loop là một control pattern trên cấu trúc Graph**. Khi một edge đưa control flow quay về node trước đó dựa trên State hiện tại, Loop được hình thành như hình trên; trọng tâm là phán đoán có đạt yêu cầu hay không, bên trong loop LLM sẽ “chấm điểm” kết quả theo yêu cầu của prompt, nếu đạt thì output, nếu không thì “trả về để viết lại”.

Loop thường gặp chủ yếu có hai loại:

1. **Fixed-count loop**: giống `for` hơn. Ví dụ “retry tối đa 3 lần”.
2. **Condition-driven loop**: giống `while` hơn. Ví dụ “chỉ cần score thấp hơn 80 thì tiếp tục revise”.

Trong scenario AI, condition-driven loop phổ biến hơn vì số iteration phụ thuộc vào content quality, tool result và external feedback. Production implementation thường còn thêm fixed upper bound: condition quyết định có tiếp tục hay không, còn round, timeout và Token budget chịu trách nhiệm force exit.

Trong engineering thực tế, cũng thường gặp **nested loop**: outer loop chịu trách nhiệm “quality iteration” (generate → review → revise), inner loop chịu trách nhiệm “tool retry” (exponential backoff retry sau khi gọi external API thất bại bên trong một node). Scope, stopping condition và counter của hai loop này độc lập: inner retry hết lượt không nên ảnh hưởng đến iteration budget của outer loop; outer loop thoát cũng không có nghĩa inner loop được retry vô hạn. Khi design nested loop, cần xác định stopping condition và safety boundary độc lập cho mỗi layer.

Tóm lại, một Loop đáng tin cậy nhất định gồm ba điều:

- Continue condition: vì sao cần thực hiện thêm một lượt.
- Exit condition: khi nào đã đủ tốt và có thể kết thúc.
- Safety boundary: số round tối đa, timeout, budget và circuit-breaker condition.

Nếu không có các constraint này, Loop rất dễ biến từ “self-correction” thành “quay vòng vô hạn”.

Quay lại ví dụ review bài viết, Loop không chỉ là “thử thêm vài lần”, mà là “kết luận review điều khiển bước nhảy tiếp theo”. Chỉ khi score chưa đạt và chưa vượt quá số round tối đa thì flow mới đi từ `ReviewNode` về `ReviseNode`; khi đạt threshold hoặc trigger boundary condition thì phải exit và đưa ra kết quả. Đến đây, loop đã trở thành một cơ chế quay lui có thể kiểm soát.

## Workflow, Graph và Loop có quan hệ thế nào?

![Tổng quan quan hệ giữa Workflow, Graph và Loop](https://oss.javaguide.cn/github/javaguide/ai/workflow/workflow-graph-loop-relation.svg)

Có thể tóm tắt quan hệ tầng của ba khái niệm bằng một câu: **Workflow là mục tiêu và quá trình, Graph là structure và carrier, Loop là control pattern trên graph.**

Tiếp tục dùng ví dụ “viết và review bài viết”:

- Khi nói “generate draft trước, sau đó review, nếu chưa đạt thì revise, cho đến khi đạt rồi output”, ta đang mô tả **Workflow**.
- Khi vẽ `generate node → check node → revise node` thành các node và connection, đồng thời cho chúng dùng chung một State, ta có **Graph**.
- Khi quy định “review không đạt thì quay về revise, cho đến khi score đạt hoặc chạm upper bound”, ta đang định nghĩa **Loop**.

Ba khái niệm này là ba góc nhìn của cùng một việc: Workflow chú trọng task goal, Graph chú trọng tổ chức structure, Loop chú trọng control quay lui.

## Triển khai bằng code

Ở phần trước đã xây dựng model khái niệm Node, Edge và State, tiếp theo sẽ xem các khái niệm này ánh xạ vào framework cụ thể thế nào. Dưới đây lấy Spring AI Alibaba Graph (Java ecosystem) và LangGraph (Python ecosystem) làm ví dụ.

### Đối chiếu khái niệm framework

Đối ứng giữa một số khái niệm then chốt trong Spring AI Alibaba và LangGraph:

- **State**: Spring AI Alibaba dùng `OverAllState` + `KeyStrategyFactory`; LangGraph dùng `TypedDict` + `Annotated[type, reducer]`
- **Replace semantic**: Spring AI Alibaba là `ReplaceStrategy`, LangGraph mặc định là như vậy
- **Append semantic**: Spring AI Alibaba dùng `AppendStrategy`, LangGraph dùng `Annotated[list, operator.add]`
- **Node**: Spring AI Alibaba là interface `NodeAction`, LangGraph là function thông thường
- **Sequential edge**: Spring AI Alibaba `addEdge(source, target)` tương ứng với LangGraph `add_edge(source, target)`
- **Conditional edge**: Spring AI Alibaba `addConditionalEdges(source, fn, map)` tương ứng với LangGraph `add_conditional_edges(source, fn)`
- **Loop**: cả hai bên đều dùng conditional edge trỏ về node trước đó, Spring AI Alibaba cung cấp thêm `LoopAgent`
- **Fixed-count loop**: cả hai bên đều có thể duy trì counter trong State, sau đó để conditional edge quyết định tiếp tục hoặc exit
- **Condition-driven loop**: cả hai bên đều có thể để conditional edge đọc score, error state hoặc external event rồi quyết định bước nhảy tiếp theo
- **Persistence**: Spring AI Alibaba dùng `MemorySaver` / `RedisSaver`..., LangGraph dùng `MemorySaver` / `SqliteSaver`
- **Human-in-the-loop**: Spring AI Alibaba dùng `interruptBefore()` + `updateState()`, LangGraph dùng `interrupt_before` + `update_state`
- **Compile và execute**: Spring AI Alibaba cần `StateGraph.compile(CompileConfig)`, LangGraph trực tiếp dùng `StateGraph.compile()`

### Ví dụ triển khai: xây dựng workflow review bài viết bằng Spring AI Alibaba

Dưới đây dùng Spring AI Alibaba Graph để triển khai workflow “generate → review → revise” xuyên suốt bài viết. Ví dụ lược bỏ import, cấu hình dependency injection và cấu hình model provider, còn node, State strategy và việc assembly Graph được giữ đầy đủ.

**Bước 1: Định nghĩa State và update strategy**

```java
// Configure State key strategies: control how each field is updated
public static KeyStrategyFactory createKeyStrategyFactory() {
    return () -> {
        HashMap<String, KeyStrategy> strategies = new HashMap<>();
        strategies.put("input", new ReplaceStrategy());          // User input
        strategies.put("messages", new AppendStrategy());        // Conversation history (append)
        strategies.put("current_draft", new ReplaceStrategy());  // Current draft (replace)
        strategies.put("review_score", new ReplaceStrategy());   // Review score (replace)
        strategies.put("review_feedback", new ReplaceStrategy()); // Review feedback
        strategies.put("iteration_count", new ReplaceStrategy()); // Iteration count
        strategies.put("output", new ReplaceStrategy());         // Final output
        strategies.put("next_node", new ReplaceStrategy());      // Routing control
        return strategies;
    };
}
```

Lưu ý `messages` dùng `AppendStrategy` (conversation history liên tục được append), còn `current_draft` dùng `ReplaceStrategy` (mỗi lần revise sẽ replace version cũ).

**Bước 2: Implement node**

```java
// Draft generation node
public static class DraftNode implements NodeAction {
    private final ChatClient chatClient;

    public DraftNode(ChatClient.Builder builder) {
        this.chatClient = builder.build();
    }

    @Override
    public Map<String, Object> apply(OverAllState state) throws Exception {
        String input = state.value("input").map(v -> (String) v).orElse("");

        String draft = chatClient.prompt()
            .user(String.format("Write an article according to the following requirements: %s", input))
            .call().content();

        return Map.of(
            "current_draft", draft,
            "next_node", "review"
        );
    }
}

// Quality review node
public static class ReviewNode implements NodeAction {
    private final ChatClient chatClient;

    public ReviewNode(ChatClient.Builder builder) {
        this.chatClient = builder.build();
    }

    private record ReviewResult(double score, String feedback) {}

    @Override
    public Map<String, Object> apply(OverAllState state) throws Exception {
        String draft = state.value("current_draft").map(v -> (String) v).orElse("");
        int count = state.value("iteration_count").map(v -> (Integer) v).orElse(0);

        String prompt = String.format(
            "Evaluate the quality of the following article, and provide a score from 0-100 and suggestions for improvement.\n" +
            "Return in JSON format: {\"score\": 85, \"feedback\": \"...\"}\n\n%s", draft);

        ReviewResult result = chatClient.prompt()
            .user(prompt)
            .call()
            .entity(ReviewResult.class);

        int nextCount = count + 1;
        String nextNode =
            (result.score() >= 80 || nextCount >= 3) ? "exit" : "revise";
        return Map.of(
            "review_score", result.score(),
            "review_feedback", result.feedback(),
            "iteration_count", nextCount,
            "next_node", nextNode
        );
    }
}

// Revision node: revise content according to review feedback
public static class ReviseNode implements NodeAction {
    private final ChatClient chatClient;

    public ReviseNode(ChatClient.Builder builder) {
        this.chatClient = builder.build();
    }

    @Override
    public Map<String, Object> apply(OverAllState state) throws Exception {
        String draft = state.value("current_draft").map(v -> (String) v).orElse("");
        String feedback = state.value("review_feedback").map(v -> (String) v).orElse("");

        String revised = chatClient.prompt()
            .user(String.format("Revise the article according to the feedback.\n\nOriginal: %s\n\nFeedback: %s", draft, feedback))
            .call().content();

        return Map.of(
            "current_draft", revised,
            "next_node", "review"
        );
    }
}

// Output node
public static class ExitNode implements NodeAction {
    @Override
    public Map<String, Object> apply(OverAllState state) throws Exception {
        String draft = state.value("current_draft").map(v -> (String) v).orElse("");
        return Map.of("output", draft);
    }
}
```

**Bước 3: Assembly Graph**

```java
public static CompiledGraph buildWorkflow(ChatModel chatModel) throws GraphStateException {
    ChatClient.Builder builder = ChatClient.builder(chatModel);

    var draft = node_async(new DraftNode(builder));
    var review = node_async(new ReviewNode(builder));
    var revise = node_async(new ReviseNode(builder));
    var exit = node_async(new ExitNode());

    StateGraph workflow = new StateGraph(createKeyStrategyFactory())
        .addNode("draft", draft)
        .addNode("review", review)
        .addNode("revise", revise)
        .addNode("exit", exit);

    // Sequential edge
    workflow.addEdge(START, "draft");

    // Conditional edge: determine routing according to the next_node field
    workflow.addConditionalEdges("draft",
        edge_async(state ->
            (String) state.value("next_node").orElse("review")),
        Map.of("review", "review"));

    workflow.addConditionalEdges("review",
        edge_async(state ->
            (String) state.value("next_node").orElse("exit")),
        Map.of(
            "revise", "revise",   // Review failed -> revise
            "exit", "exit"        // Review passed or limit reached -> output
        ));

    // Return to the review node after revision to form a loop
    workflow.addConditionalEdges("revise",
        edge_async(state ->
            (String) state.value("next_node").orElse("review")),
        Map.of("review", "review"));

    workflow.addEdge("exit", END);

    // MemorySaver only saves checkpoints within the current process
    var saver = new MemorySaver();
    var compileConfig = CompileConfig.builder()
        .saverConfig(SaverConfig.builder().register(saver).build())
        .build();

    return workflow.compile(compileConfig);
}
```

Mỗi Node chỉ xử lý một responsibility, conditional edge routing theo `next_node`, còn `iteration_count` và `review_score` được lưu trong State. `review → revise → review` tạo thành back edge; sau lần review thứ ba, dù score có đạt hay không cũng sẽ exit để tránh loop vô hạn.

`MemorySaver` trong ví dụ chỉ giữ checkpoint trong process hiện tại và cùng một Saver instance. Khi recovery cũng cần dùng thread ID hoặc session ID ổn định để định vị record. Nếu cần recovery sau khi process restart, nên đổi sang Redis hoặc database Saver, đồng thời verify behavior của checkpoint serialization, expiration và concurrent update.

> Có thể tham khảo [tài liệu chính thức Spring AI Alibaba Graph](https://java2ai.com/docs/frameworks/graph-core/quick-start/) để xem ví dụ đầy đủ hơn (bao gồm human-in-the-loop, persistence và streaming output).

## Khả năng abstraction của workflow

![So sánh workflow abstraction cao và thấp](https://oss.javaguide.cn/github/javaguide/ai/workflow/abstraction-comparison.svg)

Hình trên cho thấy workflow abstraction cao gom bốn node judgment thành một node judgment: đánh giá có đạt yêu cầu hay không. Nếu dùng abstraction thấp, khi cần giảm hoặc thêm node judgment mới, phải tốn thời gian đọc source code để tìm node tương ứng. Điểm mấu chốt của một workflow tốt là abstraction của Node, Edge và State có chịu được việc reuse và extension hay không, không liên quan nhiều đến số lượng step.

Khi design workflow, nhiều beginner dễ viết mỗi step thành một action cụ thể, chẳng hạn: gọi model để generate copy; kiểm tra độ dài title; kiểm tra tone có phù hợp không; phán đoán có cần bổ sung tài liệu không; rồi lại gọi model để revise. Cách này ngắn hạn có thể dùng, nhưng flow sẽ ngày càng vụn và khả năng reuse cũng rất kém. Cách trưởng thành hơn là abstract flow xuống một structure layer ổn định hơn:

1. **Abstract responsibility boundary của Node**: kết quả được tạo ra trong node này phải có hình dạng thế nào, nhất định phải có thông tin nào. Không phải abstract “lần này đã gọi API nào”.
2. **Abstract transfer rule của Edge**: ở State nào thì được đi đâu, khi nào kết thúc. Dùng conditional edge để biểu đạt branch và loop, thay vì viết đầy `if-else` bên ngoài Graph.
3. **Abstract thông tin State phải ghi nhớ lâu dài khi thúc đẩy task**: snapshot của ticket, kết luận review, số lần retry, error code..., để path có căn cứ.

Ví dụ trong scenario “generate và review bài viết”, thay vì design hơn mười node rời rạc để kiểm tra title có đúng đề hay không, số lượng từ có đạt yêu cầu hay không, nên abstract trước một số responsibility ổn định hơn:

- `DraftNode`: chịu trách nhiệm tạo nội dung của version hiện tại.
- `ReviewNode`: chịu trách nhiệm đánh giá kết quả hiện tại có đạt yêu cầu hay không.
- `ReviseNode`: chịu trách nhiệm revise nội dung theo feedback.
- `ExitNode`: chịu trách nhiệm output kết quả cuối cùng khi đạt condition.

![Element cốt lõi của Graph: Node, Edge, State](https://oss.javaguide.cn/github/javaguide/ai/workflow/graph-core-elements.svg)

## Khi đưa workflow vào thực tế có gặp vấn đề gì không?

Khi thực sự đưa workflow vào production, vấn đề thường không nằm ở việc “không biết vẽ Graph”, mà ở chỗ các chi tiết chưa được design trước. Dưới đây là các vấn đề thường gặp nhất trong thực tế.

### Mức độ chi tiết của State design

- Quá thô: nhét mọi thứ vào một object lớn, khó tra được ai đã sửa field nào.
- Quá chi tiết: tách field quá vụn, node nào cũng phải ghép đi ghép lại, dễ xảy ra lỗi.
- Đề xuất: chia thành vài phần theo business semantic, ví dụ “một phần cho raw input của user”, “một phần cho kết quả generate hiện tại”, “một phần cho kết luận review/score”, “một phần cho flow control (như step hiện tại, số lần retry)”.

### Stopping condition của loop

Đừng chỉ viết “nếu chưa hài lòng thì tiếp tục optimize”, mà phải xác định rõ:

- Số round tối đa là bao nhiêu?
- Score threshold là bao nhiêu?
- Khi timeout hoặc cost vượt giới hạn thì xử lý thế nào?
- Sau khi thất bại liên tiếp có cần fallback không?

### Error handling và degradation

AI Workflow không chỉ xử lý “success path”. Tool exception, model timeout, format validation thất bại, external interface rate limit đều phải có **edge rõ ràng** trên Graph: retry, degradation (như bỏ qua một tool), chuyển sang human hoặc output “best hiện tại + error description”, thay vì chỉ dựa vào `try-catch` bên ngoài để nuốt exception.

Spring AI Alibaba chia error thành bốn loại, tương ứng với các handling strategy khác nhau:

- **Transient error** (network timeout, API rate limit): dùng exponential backoff retry và đặt số lần tối đa
- **LLM recoverable error** (tool call thất bại, output format bất thường): đưa error vào State, loop quay lại để LLM xem và điều chỉnh
- **User-fixable error** (thiếu thông tin cần thiết, instruction không rõ): gọi `interruptBefore` để pause, chờ input thủ công
- **Unexpected error** (exception chưa biết): để exception bubble up, giao cho developer debug

Các strategy này khá gần với các resilience pattern trong distributed system:

- **Exponential backoff retry**: khi tool call timeout, retry theo khoảng cách tăng dần 1s, 2s, 4s và đặt tổng time limit; authentication failure phải dừng branch liên quan, authenticate lại hoặc chuyển sang human, không được bỏ qua authentication để tiếp tục thực thi
- **Circuit breaker**: khi output format của LLM thất bại liên tiếp N lần thì circuit-breaker, degrade sang template output hoặc đổi model đơn giản hơn, đừng tiếp tục lãng phí Token
- **Bulkhead isolation**: đặt concurrency upper bound độc lập cho các external API khác nhau, tránh một service chậm làm thread pool đầy
- **Compensation transaction (Saga)**: khi một step trong multi-step operation bị lỗi, thực thi rollback operation của các step đã hoàn thành theo thứ tự ngược

> Các pattern này cần được tự implement bên trong node hoặc ở middleware layer, Graph framework chỉ cung cấp execution skeleton và State management. Cách cụ thể: đóng gói logic retry và circuit-breaker trong node, persistence State thông qua field (như `retry_count`, `circuit_state`); dùng Java `Semaphore` hoặc Resilience4j cho bulkhead isolation; compensation transaction cần ghi thông tin rollback của các step đã hoàn thành vào State, sau đó design một compensation node chuyên dụng.

### Kiểm soát Token và cost

Loop sẽ tự nhiên khuếch đại Token và latency. Khi design cần suy nghĩ trước:

- Node nào bắt buộc gọi model lớn, node nào có thể thay bằng code.
- Có thể coarse filter trước rồi mới fine-tune không.
- Có cần kết thúc sớm khi đạt “đủ tốt” thay vì theo đuổi “tối ưu trên lý thuyết” không.

### Truyền data giữa các node

Truyền gì giữa các node, định nghĩa field name thế nào, structured output dùng schema nào đều nên được thống nhất sớm (ví dụ thống nhất dùng JSON Schema hoặc model Pydantic). Nếu không, khi Graph trở nên phức tạp, cost debug sẽ tăng vọt.

## Kiểm tra State và Loop trước khi release

Output của LLM trong workflow vẫn là untrusted data. Trước khi đưa vào database, frontend template, Shell command hoặc downstream tool, cần thực hiện validation và encoding tương ứng; mỗi node chỉ được cấp tool permission cần cho task hiện tại, còn các operation rủi ro cao như delete, send, payment phải được kiểm soát qua approval node.

Graph còn cần kiểm tra thêm hai loại vấn đề:

- **State pollution**: input độc hại sau khi được node xử lý có thể ghi vào field điều khiển routing của State (như `next_node`), ảnh hưởng đến routing của conditional edge tiếp theo và bỏ qua node review để đi thẳng đến output. Phòng thủ: whitelist validation cho field điều khiển routing trong State.
- **Loop amplification attack**: input độc hại được tạo để `ReviewNode` luôn trả score thấp, khiến Loop chỉ thoát khi đạt maximum round và tiêu hao lượng lớn Token. Phòng thủ: ngoài upper bound của `iteration_count`, thêm Token consumption budget làm safety boundary độc lập.

Cuối cùng, dùng replay test để cover exit bình thường, đạt iteration limit, tool timeout, human interrupt, authentication failure và checkpoint recovery. Framework API sẽ thay đổi, nhưng responsibility của Node, flow hợp lệ của Edge và update rule của State nên được cố định trong test.

## Điểm cần chuẩn bị cho phỏng vấn

**Câu hỏi thường gặp**:

1. **Vì sao AI system cần workflow?** → Output của LLM không chắc chắn, cần dynamic decision, automatic correction và convergence có thể kiểm soát
2. **Quan hệ giữa Workflow, Graph và Loop là gì?** → Workflow là mục tiêu và quá trình, Graph là structure và carrier, Loop là control pattern trên graph
3. **Graph Loop và Agent Loop khác nhau thế nào?** → Agent Loop là execution engine cấp cao nhất của Agent (vòng lặp reasoning → action → observation), Graph Loop là control pattern quay lui bên trong Graph (một subset node cụ thể iteration và revise thông qua back edge), hai loại có thể nest
4. **Loop ngăn infinite loop thế nào?** → Ba yếu tố: continue condition, exit condition và safety boundary (maximum round + timeout + Token budget)
5. **Chọn update strategy cho State thế nào?** → Field single-value dùng Replace, field tích lũy dùng Append, field được ghi song song bắt buộc dùng Reducer
6. **Conditional edge và dynamic routing khác nhau thế nào?** → Candidate set của conditional edge được xác định khi design và lựa chọn tại runtime; candidate set của dynamic routing chỉ được xác định tại runtime; thực tế đây là một spectrum liên tục
7. **Hiểu abstract design của Graph thế nào?** → Node abstract responsibility boundary (tạo ra gì), Edge abstract transfer rule (khi nào đi đâu), State abstract thông tin bắt buộc phải ghi nhớ lâu dài

**Chuẩn bị cho câu hỏi đào sâu**:

- Làm thế nào để recovery sau khi workflow bị interrupt? (persistence + checkpoint mechanism)
- Xử lý error bên trong node thế nào? (transient error retry, LLM recoverable error quay lại loop, user-fixable error chuyển sang human, unexpected error bubble up)
- Cách implement loop của Spring AI Alibaba và LangGraph khác nhau thế nào? (loại trước có thể dùng conditional edge trỏ về trước hoặc `LoopAgent`, loại sau cần tự duy trì counter)
- Workflow có những security risk đặc thù nào? (State pollution ảnh hưởng routing, loop amplification attack tiêu hao Token)

## Tổng kết

Workflow mô tả quá trình hoàn thành task, Graph dùng Node, Edge và State để tổ chức quá trình thành structure có thể thực thi, còn Loop cho phép các node cụ thể quay về step trước để tiếp tục revise theo condition. Chúng cung cấp cho việc generate, evaluate và tool call có chứa LLM một cách truyền State và xử lý fault rõ ràng hơn so với một Prompt dài duy nhất.

Khi đưa vào production, trước hết cần định nghĩa responsibility và update rule của State, sau đó viết rõ conditional edge, exit condition, maximum round, timeout và Token budget. Mọi data đi vào routing, database, template hoặc external tool đều phải được validate; retry, degradation, human interrupt và checkpoint recovery cũng nên được đưa vào test coverage, tránh để loop liên tục khuếch đại cost trước input lỗi hoặc dependency bất thường.
