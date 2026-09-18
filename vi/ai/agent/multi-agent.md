---
title: "Thiết kế hệ thống phối hợp Multi-Agent: chia nhiệm vụ, chia sẻ state, xử lý xung đột và khôi phục khi lỗi"
description: "Thông qua các bối cảnh như AgentInvest, nghiên cứu kỹ thuật, chuyển tiếp chăm sóc khách hàng và review code, giải thích sự khác biệt, cách orchestration, communication, state, xung đột, khôi phục và đánh giá giữa Multi-Agent và Prompt Chain."
category: AI
tag:
  - AI Agent
  - Multi-Agent
  - System design
head:
  - - meta
    - name: keywords
      content: Multi-Agent,Multi-Agent,multi-agent stock analysis system,AgentInvest,AgentScope Java,ReActAgent,chia nhiệm vụ,chia sẻ state,xử lý xung đột,khôi phục khi lỗi,A2A
---

<!-- @include: @article-header.snippet.md -->

Khi xây dựng hệ thống phân tích cổ phiếu Multi-Agent AgentInvest, tôi giao nhiệm vụ cho bốn role. Researcher hoàn thành nghiên cứu nền tảng trước, Technical Analyst và Sentiment Analyst bổ sung song song, Investment Manager tổng hợp sau cùng. Bên trong mỗi role dùng ReAct để gọi tool, còn pipeline bên ngoài kiểm soát thứ tự, concurrency và điều kiện lỗi.

Sau khi các role chạy, vấn đề engineering mới thực sự xuất hiện. Chia subtask đến mức nào, hai Worker có thể đồng thời ghi state hay không, một nhánh timeout thì có tiếp tục không, đều ảnh hưởng trực tiếp đến kết quả cuối. Khi interviewer hỏi sâu về dự án Multi-Agent, họ thường cũng sẽ tiếp tục xoay quanh các vấn đề này.

[Khái niệm cốt lõi của AI Agent](./agent-basis.md#multi-agent) đã giới thiệu các collaboration pattern như Orchestrator-Subagent và Peer-to-Peer. Bài viết này tiếp tục thảo luận cách xử lý task, state, xung đột và khôi phục khi lỗi sau khi các pattern này được đưa vào dự án.

Ngoài AgentInvest, bài viết cũng đan xen các ví dụ về nghiên cứu kỹ thuật, chuyển tiếp chăm sóc khách hàng và review code.

## Multi-Agent khác gì Multi-stage Prompt Chain?

Prompt Chain quản lý các bước xử lý, còn Multi-Agent quản lý các executor có thể độc lập hoàn thành subgoal. Hai loại này có thể cùng xuất hiện: bên ngoài dùng fixed Workflow, bên trong mỗi node lại chạy Agent riêng.

| Hạng mục so sánh | Multi-stage Prompt Chain                                 | Multi-Agent                                                                   |
| ---------------- | -------------------------------------------------------- | ----------------------------------------------------------------------------- |
| Execution unit   | Node xử lý trong một Workflow                            | Agent có thể độc lập hoàn thành subgoal                                       |
| Cách làm việc    | Node xử lý input theo logic định trước và tạo output     | Agent có thể chọn tool, quan sát kết quả và lặp thực thi                      |
| Cách kiểm soát   | Workflow phụ trách chuyển node và chuyển state           | Có thể dùng orchestration cố định hoặc model dynamic delegation               |
| Context          | Thường truyền dọc theo processing chain                  | Có thể tách context, tool, permission và state theo Agent                     |
| Runtime state    | Chủ yếu quan tâm chain đã chạy đến node nào              | Còn phải phân biệt task, state, số lần thử và kết quả bàn giao của từng Agent |
| Bàn giao kết quả | Output của node thường là input của node sau             | Có thể truyền message, chia sẻ state hoặc bàn giao kết quả có cấu trúc        |
| Xử lý lỗi        | Retry node lỗi hoặc khôi phục từ checkpoint của Workflow | Còn phải quyết định role nào có thể degrade, bỏ qua hoặc chạy lại             |

![Sự khác biệt giữa Multi-Agent và Multi-stage Prompt Chain](https://oss.javaguide.cn/github/javaguide/ai/agent/multi-agent-vs-prompt-chain.webp)

Nếu bốn role của AgentInvest chỉ là bốn lần gọi model cố định, dùng chung một context và một bộ tool, mỗi bước chỉ xử lý văn bản của bước trước, thì đó vẫn là một Multi-stage Prompt Chain.

Mỗi role trong dự án là một `ReActAgent` độc lập, có Prompt, Toolkit, `maxIters` và context riêng. Researcher có thể tra cứu giá thị trường, báo cáo nghiên cứu, tin tức và dữ liệu tài chính; Technical Analyst tập trung vào K-line và technical indicator; Sentiment Analyst tra cứu market sentiment; Investment Manager đọc `<core_thesis>` từ upstream để đưa ra đánh giá tổng hợp. Workflow bên ngoài quản lý execution process của bốn Agent độc lập, không chỉ truyền kết quả giữa bốn đoạn văn bản.

Chạy song song chỉ cho biết task được thực thi thế nào. Multi-Agent có thể bàn giao tuần tự, còn Prompt Chain cũng có thể có nhánh song song.

## Nếu thiết kế một hệ thống Multi-Agent, bạn sẽ cân nhắc vấn đề nào trước?

Trước hết xác định mỗi role phụ trách gì, ai kiểm soát bước tiếp theo, bàn giao kết quả ra sao và xử lý thế nào sau khi lỗi; cuối cùng mới quyết định cần bao nhiêu Agent và task nào có thể chạy song song.

| Vấn đề cần xác định             | Cần cân nhắc cụ thể gì                                                                            | Một ví dụ thường gặp                                                                          |
| ------------------------------- | ------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| Mỗi Agent phụ trách gì?         | Có cần tách biệt subgoal, Prompt, tool, context và permission không                               | Nghiên cứu kỹ thuật có thể tách thành truy vấn tài liệu, phân tích dữ liệu và review kết luận |
| Ai quyết định bước tiếp theo?   | Flow do code kiểm soát hay model dynamic chọn role và task                                        | Quy trình review cố định giao cho code, hướng truy vấn chưa rõ giao cho model                 |
| Nhiều Agent phối hợp thế nào?   | Tuần tự, song song, routing, handoff, loop hay kết hợp nhiều cách                                 | Nhiều nhánh truy vấn tài liệu chạy song song, Reviewer review chung sau khi đủ tài liệu       |
| Agent bàn giao gì cho nhau?     | Truyền toàn bộ conversation, chia sẻ state hay chỉ truyền structured result và evidence cần thiết | Research Agent bàn giao conclusion, citation và Artifact URI thay vì toàn bộ conversation     |
| Ghi state và xử lý lỗi thế nào? | Có ghi riêng state của role không; sau timeout thì retry, bỏ qua, degrade hay terminate           | Giữ các kết quả khác sau khi một nhánh truy vấn timeout và đánh dấu rõ phần thiếu             |
| Agent được deploy ở đâu?        | Chạy trong cùng process hay qua network, cross-service, cross-team hoặc cross-security domain     | Role local có thể gọi trực tiếp, role remote cần identity, protocol và task state             |

Lấy nghiên cứu kỹ thuật làm ví dụ: Workflow bên ngoài có thể cố định thành bốn stage: lập kế hoạch, truy vấn tài liệu, phân tích và review. Trong stage truy vấn tài liệu, Agent chọn search term, data source và tool; khi cần có thể thêm hướng truy vấn mới. Các role chỉ bàn giao conclusion, citation và Artifact, không cần chia sẻ toàn bộ chat history.

## Khi nào nên dùng Multi-Agent?

Thông thường tôi sẽ bắt đầu bằng Single Agent hoặc fixed Workflow. Khi một Agent đã hoàn thành task với cost và latency chấp nhận được, tách thành nhiều role chỉ làm tăng khối lượng orchestration, bàn giao context, merge result và recovery khi lỗi.

Nếu các khâu khác nhau cần tool, permission và context rõ rệt khác nhau, hoặc task có thể tách thành vài hướng tương đối độc lập, hãy cân nhắc Multi-Agent. Mỗi role cũng phải có output rõ ràng, chẳng hạn structured conclusion, citation, code Diff hoặc test report. Nếu các role chỉ bàn giao vài đoạn free text, về sau thường sẽ phải tốn nhiều công sức ghép kết quả và định vị vấn đề.

Hệ thống [Multi-Agent Research](https://www.anthropic.com/engineering/multi-agent-research-system) được Anthropic công khai áp dụng thiết kế này. Main Agent chia câu hỏi thành nhiều hướng nghiên cứu, Subagent lần lượt tìm kiếm, sau đó Main Agent tổng hợp. Các nhánh có thể khám phá độc lập, phù hợp với parallel; nếu các nhánh thường xuyên phụ thuộc vào intermediate conclusion của nhau, hoặc phải liên tục đồng bộ toàn bộ context, communication cost sẽ nhanh chóng triệt tiêu lợi ích của việc tách nhỏ.

![Kiến trúc hệ thống Multi-Agent](https://oss.javaguide.cn/github/javaguide/ai/agent/agent-multi-agent-arch.png)

Trong task thay đổi code, Coding Agent phụ trách sửa file và bàn giao Diff, Testing Agent chạy test và trả về thông tin lỗi, Reviewer kiểm tra theo requirement và Diff. Ba role dùng tool và acceptance criteria khác nhau, sau khi tách thì task của mỗi role đều rõ ràng. Nếu chúng chỉ đọc cùng một Diff rồi lần lượt viết các đoạn summary tương tự nhau, thứ tăng lên chỉ là số lần gọi model.

Trước khi deploy, vẫn phải dùng cùng một batch task để chạy comparison test, so sánh quality, thời gian, Token và failure rate giữa Single Agent và Multi-Agent. Không có lợi ích đo được thì trước tiên giữ implementation đơn giản hơn.

## Bốn Agent trong AgentInvest phối hợp thế nào?

Lấy task phân tích cổ phiếu của AgentInvest làm ví dụ, user đưa ra câu hỏi:

> Hãy phân tích xem hiện tại còn nên mua Kweichow Moutai không.

Model không thể trả lời câu hỏi này chỉ bằng kiến thức chung. Hệ thống cần truy vấn market quote theo thời gian thực, research report, financial indicator, K-line và market sentiment, đồng thời kết hợp thông tin user có đang nắm giữ cổ phiếu hay không để đưa ra đề xuất. AgentInvest thực thi theo thứ tự sau.

1. Researcher trước tiên truy vấn market quote, research report, tin tức và dữ liệu tài chính, bàn giao kết luận nghiên cứu nền tảng cùng `<core_thesis>`.
2. Sau khi Researcher hoàn thành, Technical Analyst truy vấn K-line và technical indicator, Sentiment Analyst truy vấn market sentiment; hai role chạy song song.
3. Investment Manager đọc upstream argument, sau đó kết hợp portfolio của user, historical memory và strategy Skill để tạo đề xuất cuối.

Pipeline execution strategy tùy chỉnh `AgentScopePipelineExecutionStrategy` của project kiểm soát thứ tự stage bên ngoài. Chỉ bên trong role mới giao cho ReAct để model tự xác định còn thiếu dữ liệu gì, gọi tool nào và có tiếp tục phân tích sau khi nhận kết quả hay không. Cách này vừa bảo đảm pipeline research ổn định, vừa không biến mỗi role thành một Prompt Chain hoàn toàn cố định.

![Sub-agent chia task, tách context](https://oss.javaguide.cn/github/javaguide/ai/context-engineering/sub-agent-task-splitting-context-isolation%20.png)

Để bước vào stage Investment Manager, phải thỏa mãn hai điều kiện. Researcher đã tạo core argument, và ít nhất một trong Technical Analyst hoặc Sentiment Analyst thành công. Khi upstream result rõ ràng không đủ, hệ thống bỏ qua Investment Manager ngay và trả lỗi qua SSE, tránh để model cố ghép một investment advice trông có vẻ đầy đủ.

Researcher bắt buộc chạy trước, Investment Manager bắt buộc chạy cuối; chỉ hai role ở giữa là không phụ thuộc nhau và chạy song song. Role được xác định bởi thiết kế Multi-Agent, còn thứ tự thực thi do DAG kiểm soát.

## Multi-Agent nên chọn orchestration pattern nào?

Orchestration pattern phụ thuộc vào ai nắm quyền kiểm soát, role được xác định khi nào, task phụ thuộc nhau ra sao và ai trực tiếp tương tác với user.

| Pattern             | Cách kiểm soát                                                              | Bối cảnh phù hợp                                                                         | Chi phí chính                                        |
| ------------------- | --------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| Sequential          | Code gọi nhiều Agent theo thứ tự cố định                                    | Step xác định, dependency trước sau rõ                                                   | Latency cộng dồn, ít linh hoạt                       |
| Parallel            | Code phân phát rồi merge các task độc lập song song                         | Review nhiều chiều, truy vấn độc lập, voting                                             | Merge state và xử lý partial failure                 |
| Router              | Rule hoặc model route request đến một hoặc nhiều expert                     | Business domain rõ, phân loại tương đối ổn định                                          | Route sai sẽ chọn nhầm expert                        |
| Supervisor/Manager  | Main Agent dynamic gọi Subagent và giữ quyền kiểm soát cuối                 | Subtask chưa thể xác định hoàn toàn từ trước, cần một câu trả lời thống nhất             | Main Agent thành bottleneck, coordination cost cao   |
| Handoff             | Agent hiện tại bàn giao quyền kiểm soát và context cần thiết cho Agent khác | Chuyển tuyến chăm sóc khách hàng, conversation theo stage, expert phục vụ user trực tiếp | Khó duy trì chat history và permission boundary      |
| Evaluator-Optimizer | Generator và evaluator lặp lại, cải tiến qua nhiều vòng                     | Có scoring criteria rõ và iteration thực sự cải thiện kết quả                            | Dễ polish vô hạn, cần stopping condition             |
| Group Chat          | Agent chia sẻ conversation, lần lượt phát biểu theo round hoặc Selector     | Discussion, review, task cần sửa lẫn nhau                                                | Context dễ phình to, khó xác định stopping condition |
| Debate              | Nhiều Agent đưa ra, chất vấn và sửa quan điểm qua nhiều vòng                | Kết luận có thể kiểm tra bằng evidence, bất đồng có giá trị                              | Dễ tranh luận lặp lại, không bảo đảm đúng hơn        |
| Event-driven/Peer   | Agent quyết định action tiếp theo theo message và local rule                | Open collaboration cross-service, cross-team                                             | Phức tạp nhất về consistency, security và debug      |

AgentInvest dùng orchestration hỗn hợp **Sequential + Parallel + Aggregator**. Researcher và Investment Manager tạo thành thứ tự trước sau; Technical Analyst và Sentiment Analyst ở giữa chạy song song; Investment Manager phụ trách merge cuối.

Ở đây không dùng Supervisor để dynamic quyết định số lượng role, cũng không để Agent nào Handoff user session cho Agent khác. Các stage chính của stock analysis tương đối cố định, dùng code kiểm soát flow bên ngoài ổn định hơn; phần cần dynamic judgment được xử lý trong ReAct Loop của từng role.

Các pattern này có thể kết hợp. Sau khi Router chọn expert, expert vẫn có thể gọi Subagent; Supervisor cũng có thể hoàn thành preparation task trước rồi mới parallel dispatch Worker.

### Role nên được tách static hay dynamic delegation?

Static split nghĩa là role, dependency và execution path đã được xác định trước khi chạy. Fixed order, fixed parallel branch và conditional DAG đều thuộc loại này. Ưu điểm là dễ test, kiểm soát timeout, budget và partial failure tốt hơn; phù hợp với task có business step tương đối ổn định.

Dynamic delegation là Manager hoặc Supervisor căn cứ vào vấn đề hiện tại và intermediate result để quyết định gọi Agent nào, tách bao nhiêu subtask và có tiếp tục khám phá hay không. Multi-Agent Research của Anthropic thuộc pattern này: mỗi vấn đề cần điều tra các hướng khác nhau, rất khó vẽ trước một DAG cố định.

Hai cách có thể dùng kết hợp. Ví dụ, outer flow cố định ba stage thu thập tài liệu, phân tích và review; trong stage thu thập tài liệu, Main Agent tạo Subagent dựa trên câu hỏi. Review và publish gate vẫn do fixed flow kiểm soát, còn hướng research có thể mở rộng lúc runtime.

![Sự khác biệt giữa fixed DAG và dynamic delegation của Multi-Agent](https://oss.javaguide.cn/github/javaguide/ai/agent/multi-agent-static-vs-dynamic-orchestration.webp)

## Làm thế nào tách subtask thành execution contract?

Chỉ viết “bạn là Technical Research Agent” trong Prompt thì role vẫn không biết cần tra cứu gì, có thể dùng dữ liệu nào và thế nào mới được tính là hoàn thành. Khi orchestrator dispatch task, phải đồng thời đưa ra subgoal và success criteria, chỉ rõ upstream task dependency, tool và permission khả dụng, đồng thời quy định output Schema và nơi lưu artifact. Time, Token, số lần gọi và failure strategy cũng thuộc task contract, do runtime thực thi.

`AgentTask` dưới đây minh họa một task contract đơn giản.

```java
public record AgentTask(
        String taskId,
        String parentTaskId,
        AgentRole role,
        TaskContext context,
        List<AgentRole> dependencies,
        Set<String> allowedTools,
        String outputSchema,
        int maxIters,
        Duration modelTimeout,
        Duration toolTimeout,
        Instant deadline,
        int maxAttempts) {
}
```

Project thực tế còn phải bổ sung task version, idempotency key, success criteria và artifact location. Runtime bắt buộc thực thi `allowedTools`, timeout và số lần retry; model chỉ nhận thông tin cần để hoàn thành subtask hiện tại.

### Làm thế nào xác định subtask có thể chạy song song?

Lấy nghiên cứu kỹ thuật làm ví dụ: sau khi xác định research question, “tra cứu official documentation” và “tra cứu paper cùng benchmark” có thể bắt đầu đồng thời. Input của hai bên đã sẵn sàng và cũng không cần đọc intermediate conclusion của nhau. Việc đối chiếu conclusion phải đợi tài liệu của cả hai bên đầy đủ, nên chỉ có thể đặt về sau.

Parallel branch cũng phải tránh cùng một authoritative field. Hai Research Agent lần lượt submit material list của mình, merge node viết final report; các external side effect như gửi notification, sửa code cũng giao cho một executor duy nhất, hoặc dùng idempotency và unique constraint để bảo vệ. Một branch lỗi thì wait, degrade hay terminate cũng phải được ghi vào success criteria trước khi chạy.

Trong Java có thể dùng thread pool, virtual thread hoặc structured concurrency để thực thi các branch này; chọn cách nào không phải trọng điểm. Orchestration layer vẫn phải giới hạn max concurrency và đặt chung Deadline cho toàn stage; khi hết hạn thì cancel task chưa hoàn thành, giữ kết quả đã hoàn thành, rồi dựa trên success gate để quyết định có vào downstream hay không.

### Làm thế nào tránh role làm trùng và thiếu kết quả?

Phạm vi trách nhiệm phải được áp vào cả task lẫn tool. Trong bối cảnh code change, Coding Agent phụ trách sửa workspace, Testing Agent chỉ chạy test và trả về lỗi, Reviewer đọc Diff, test result và rule checklist. Chỉ đổi tên role mà không giới hạn tool và output thì các Agent vẫn sẽ kiểm tra trùng một việc.

Mỗi role cũng cần một delivery protocol rõ ràng. `RoleResult` thông dụng ít nhất có thể gồm conclusion, evidence, data timestamp, Artifact URI, missing item và confidence boundary; coordination layer phụ trách validate required field. Structured output giải quyết vấn đề format và handoff, còn conclusion có đáng tin hay không vẫn phải đánh giá cùng evidence và freshness.

AgentInvest hiện dùng `<core_thesis>` để truyền core argument, đây là cách nhẹ và chi phí cải tạo thấp; nó cũng có thể làm mất data timestamp và căn cứ quan trọng. Khi cần auditability mạnh hơn, nên từng bước thay bằng structured result, không tiếp tục mở rộng các fallback rule dạng natural language.

## Agent nên communication và chia sẻ state thế nào?

Agent không mặc định phải dùng chat message. Fixed Workflow trong process thường bàn giao result qua parameter, return value và shared state; chỉ khi cross-service hoặc cần negotiation liên tục mới thêm message và remote protocol.

| Communication method      | Cách làm                                                          | Phù hợp với trường hợp nào                                       |
| ------------------------- | ----------------------------------------------------------------- | ---------------------------------------------------------------- |
| Parameter và return value | Orchestrator gọi Agent, dùng upstream result làm downstream input | Fixed Workflow trong process, quan hệ đơn giản                   |
| Shared state/blackboard   | Agent đọc shared state, chỉ update field do mình phụ trách        | Graph orchestration, nhiều node cần từng bước hoàn thiện result  |
| Message                   | Gửi point-to-point hoặc broadcast đến Group Chat                  | Agent cần negotiation, handoff hoặc async collaboration liên tục |
| Structured result         | Bàn giao conclusion, report hoặc file reference có Schema         | Result lớn, cần validate, reuse và audit                         |
| Remote Agent protocol     | Dùng protocol như A2A để discover Agent, truyền task và artifact  | Cross-process, cross-team hoặc cross-security domain             |

Communication mechanism càng phức tạp thì authentication, retry, message order và duplicate delivery càng khó xử lý. Nếu có thể bàn giao bằng method call thì không cần trước tiên đổi thành message system.

Shared state chỉ lưu thông tin cần cho collaboration giữa các role. Tool record và ReAct progress của mỗi Agent giữ trong session state riêng; user request, tenant, resource và permission truyền rõ ràng qua `TaskContext`. Khi role hoàn thành, nó ghi conclusion, evidence summary, missing item và Artifact URI vào state slot do mình phụ trách; orchestrator ghi riêng role đã start, complete, fail hay timeout.

Raw material như webpage, log, query result và code Diff có thể giữ trong context hiện tại của role hoặc Artifact store. SSE, WebSocket chỉ phụ trách đẩy progress đến frontend, không đảm nhiệm lưu task state.

Business context nên được truyền rõ ràng cho Agent và tool. Dựa vào `ThreadLocal` để implicit đọc task data dễ làm lẫn task sau concurrency, async hoặc remote call, đồng thời khó nhìn ra từ method signature tool đã dùng những thông tin nào.

### Làm thế nào kiểm soát kích thước shared state?

Trong task nghiên cứu kỹ thuật, Research Agent không cần copy toàn bộ webpage đã crawl, search process và tool log cho Reviewer. Nó có thể lưu raw material vào Artifact store, chỉ bàn giao conclusion summary, citation, data timestamp và Artifact URI. Khi cần verify một conclusion, Reviewer đọc raw evidence theo URI.

Summary không được tách khỏi evidence. Với thông tin có tính thời điểm cao như price, version, regulation và experimental data, còn phải giữ collection time, source và key field.

### Runtime state của Multi-Agent cần ghi lại gì?

```java
public record AgentRunState(
        String runId,
        String subject,
        RunStatus status,
        Map<AgentRole, RoleState> roles,
        Map<AgentRole, ArtifactRef> artifacts,
        BudgetUsage usage,
        List<FailureRecord> failures,
        ArtifactRef finalOutput) {
}

public record RoleState(
        AgentRole role,
        TaskStatus status,
        int attempt,
        String providerId,
        String modelName,
        Instant startedAt,
        Instant updatedAt,
        FailureRecord lastFailure) {
}
```

Đây là cấu trúc minh họa đã giản lược. Session state của Single Agent, shared business state và outer Workflow state phải được lưu riêng. Session store ghi model conversation và tool progress, còn Workflow state ghi task đã chạy đến role nào; hai loại không thể thay thế lẫn nhau.

Sau khi tách role thành service độc lập, bổ sung `ownerAgentId`, `version`, `leaseUntil` và `fencingToken` để xử lý task takeover. Các field này thuộc distributed Worker layer; Multi-Agent trong process không cần sao chép từ đầu.

## A2A giải quyết được gì và không giải quyết được gì?

Khi nhiều role chạy trong cùng process, gọi method trực tiếp thường đơn giản hơn. Sau khi Agent được deploy độc lập, do các team khác nhau maintain hoặc cần gọi cross-security domain, mới cân nhắc dùng A2A để thống nhất capability discovery, task state và artifact delivery.

Một A2A collaboration thường bắt đầu bằng việc đọc Agent Card. Caller trước tiên xác nhận capability, interface và authentication requirement của remote Agent, sau đó tạo Task có unique ID. Hai bên bổ sung task information qua Message; khi hoàn thành thì dùng Artifact để bàn giao document hoặc structured result, còn Context liên kết các task và message liên quan.

[A2A 1.0](https://a2a-protocol.org/latest/announcing-1.0/) có `TaskState`, ngoài `TASK_STATE_UNSPECIFIED` biểu thị trạng thái unknown hoặc unspecified, còn gồm `TASK_STATE_SUBMITTED`, `TASK_STATE_WORKING`, `TASK_STATE_INPUT_REQUIRED`, `TASK_STATE_AUTH_REQUIRED`, `TASK_STATE_COMPLETED`, `TASK_STATE_FAILED`, `TASK_STATE_CANCELED` và `TASK_STATE_REJECTED`. Trong đó, `INPUT_REQUIRED` và `AUTH_REQUIRED` đều là interrupted state, chờ external information bổ sung rồi mới tiếp tục. Cách này phù hợp với long-running task hơn việc chỉ trả về “success/failure”.

A2A giải quyết việc remote Agent discover nhau, exchange message, track task và delivery artifact. Nó không quyết định execution topology thay cho application và cũng không tự giải quyết internal consistency. Hai Worker cùng update task thì sao, sau timeout ai takeover, làm thế nào tránh notification hoặc refund bị execute trùng, vẫn cần business system tự xử lý.

## Xử lý concurrency conflict trong Multi-Agent thế nào?

Khi thấy hai Agent đưa ra kết quả khác nhau, không thể đơn giản chọn một kết quả ghi đè kết quả kia. Trước tiên phải phân biệt chúng gặp loại conflict nào.

| Loại conflict        | Một ví dụ thường gặp                                             | Cách xử lý                                                     |
| -------------------- | ---------------------------------------------------------------- | -------------------------------------------------------------- |
| State write conflict | Hai Research Agent đồng thời ghi đè field `sources` của report   | Tách slot theo role, append collection hoặc dùng version/CAS   |
| Opinion conflict     | Hai Agent dựa trên source khác nhau đưa ra conclusion trái ngược | Giữ source, time và scope, giao Aggregator phán đoán           |
| Ownership conflict   | Worker cũ và Worker takeover đồng thời submit result             | Lease + version/CAS + Fencing Token                            |
| Side-effect conflict | Hai Agent gửi notification hoặc tạo refund trùng nhau            | Single executor, idempotency key và business unique constraint |

### Merge parallel state thế nào?

Parallel Agent lần lượt update state slot của mình; result và error cũng lưu theo role. Các counter như Token và time có thể được aggregate qua event, còn final report và `finalOutput` chỉ cho phép merge node ghi. Frontend event mang theo role và sequence number để hiển thị, không được ngược lại sửa task state.

Nếu hai Agent đều có thể ghi đè cùng một `finalOutput`, thread-safe container chỉ bảo đảm quá trình ghi không hỏng, chứ không xác định được business result nào nên giữ. Lưu result theo role, sau đó để một merge node duy nhất ghi authoritative field mới có thể tránh last writer trực tiếp ghi đè các branch khác.

### Xử lý conflict giữa factual conclusion của nhiều Agent thế nào?

“Conflict” trong stock analysis không nhất thiết là ai tính sai. Technical Analyst có thể dựa vào K-line 60 ngày để nhận định trend thiên bullish, trong khi Sentiment Analyst phát hiện short-term negative sentiment đang tăng. Hai conclusion tập trung vào time range và data source khác nhau, không thể chỉ voting để quyết định nghe ai.

Nếu sau này cần tăng auditability của conclusion, có thể tiếp tục structured hóa các opinion quan trọng.

```json
{
  "claimId": "claim-17",
  "subject": "600519",
  "predicate": "technical-trend",
  "value": "BULLISH",
  "scope": "daily-kline-60d",
  "sourceTool": "get_technical_indicators",
  "observedAt": "2026-08-15T10:00:00+08:00",
  "evidenceStatus": "VERIFIED"
}
```

`scope` dùng để chỉ conclusion đến từ daily line hay minute line, `observedAt` ghi data timestamp, `sourceTool` cho biết căn cứ đến từ tool nào. Khi Investment Manager merge, nó có thể phân biệt “technical trend trung hạn thiên bullish” và “sentiment ngắn hạn thiên bearish”, không cần ép chúng thành cùng một hướng.

Khi thiếu dữ liệu, final advice nên giảm mức certainty. ReAct có thể giảm trường hợp trả lời khi chưa tra cứu dữ liệu, nhưng không bảo đảm investment conclusion luôn đúng.

### Vì sao Lease còn phải đi kèm Fencing Token?

Khi task trong process có thể trực tiếp cancel và wait, thường không cần distributed Lease. Chỉ sau khi role được tách thành remote Worker mới gặp vấn đề instance cũ và instance takeover cùng trả kết quả.

Giả sử Worker A nhận task lease rồi bị pause trong thời gian dài. Lease hết hạn, hệ thống giao task cho Worker B. Lúc này A khôi phục và tiếp tục ghi; nếu downstream chỉ kiểm tra “A từng giữ lock”, result cũ vẫn có thể ghi đè result mới của B.

Giả sử `fencingToken` A nhận được là 7, B takeover rồi nhận 8. State store ghi 8 là version hiện được chấp nhận; sau khi A khôi phục vẫn submit kèm 7, write condition không thỏa mãn và late result bị reject.

Lease chủ yếu giải quyết việc Owner mất kết nối không chiếm task vĩnh viễn; Fencing Token hoặc version validation giải quyết late write sau khi Owner cũ khôi phục. Chỉ đặt TTL cho lock không tự động bảo vệ được mọi external resource.

## Nên model task lifecycle thế nào?

Role state phải do outer execution strategy quản lý, không thể để model tự điền. Orchestrator cần dựa trên state của dependency task để quyết định downstream có thể chạy hay không. Dưới đây là một thiết kế state machine thông dụng.

| State              | Ý nghĩa                                     | Các state kế tiếp chính được phép                                |
| ------------------ | ------------------------------------------- | ---------------------------------------------------------------- |
| `PENDING`          | Đã tạo, dependency chưa đủ                  | `READY`, `CANCELED`                                              |
| `READY`            | Có thể được schedule                        | `RUNNING`, `CANCELED`                                            |
| `RUNNING`          | Role đang thực thi                          | `WAITING_INPUT`, `SUCCEEDED`, `RETRYABLE_FAILED`, `FINAL_FAILED` |
| `WAITING_INPUT`    | Chờ user input, approval hoặc authorization | `READY`, `CANCELED`, `FINAL_FAILED`                              |
| `RETRYABLE_FAILED` | Đã xác nhận là lỗi có thể retry             | `READY`, `FINAL_FAILED`                                          |
| `SUCCEEDED`        | Output đã lưu và qua basic validation       | Terminal state                                                   |
| `FINAL_FAILED`     | Không thể tự khôi phục                      | Terminal state hoặc mở lại thủ công                              |
| `CANCELED`         | Bị user hoặc system cancel                  | Terminal state                                                   |

Trong pipeline single-process hiện tại, lưu state theo role là đủ để hoàn thành phần control chính. Sau này khi tách thành distributed service, lúc submit result còn phải đồng thời validate task state, Owner, Fencing Token và version:

```sql
UPDATE agent_task
SET status = 'SUCCEEDED',
    artifact_uri = :artifactUri,
    version = version + 1
WHERE task_id = :taskId
  AND status = 'RUNNING'
  AND owner_agent_id = :ownerAgentId
  AND fencing_token = :fencingToken
  AND version = :expectedVersion;
```

Affected row count bằng 0 nghĩa là task đã bị takeover, cancel hoặc có concurrent update. Worker không thể tiếp tục “ghi bù”, mà phải đọc lại state.

## Khôi phục thế nào sau khi Multi-Agent lỗi?

Trong hệ thống Multi-Agent, sau lỗi không thể retry đồng loạt. Input không đầy đủ, thiếu permission, search API timeout và result thiếu evidence có cách xử lý hoàn toàn khác nhau:

| Loại lỗi                      | Một ví dụ thường gặp                                              | Action đề xuất                                        |
| ----------------------------- | ----------------------------------------------------------------- | ----------------------------------------------------- |
| Input/contract error          | Thiếu target resource, output không phù hợp Schema                | Sửa một lần hoặc fail ngay, không retry network       |
| Permission/business rejection | Customer service Agent không có quyền refund                      | Terminate call hoặc chờ user/admin authorize          |
| Transient failure             | Search API connection reset, rate limiting ngắn hạn, 5xx tạm thời | Bounded retry, backoff kèm jitter                     |
| Call timeout                  | Model hoặc tool vượt call budget lần này                          | Terminate call hiện tại, xử lý theo strategy của role |
| Role timeout                  | Một research branch vượt Deadline của toàn stage                  | Cancel task chưa hoàn thành, giữ successful result    |
| Quality failure               | Report thiếu citation, required field hoặc artifact có thể verify | Sửa một lần, degrade hoặc bỏ qua downstream           |
| Deterministic code error      | Parse event thất bại, state transition bất hợp lệ                 | Fail fast, alert, sửa code                            |

Recovery action phải được đặt ở đúng layer chịu trách nhiệm. Role executor xử lý model, tool và số lần iteration; orchestrator quyết định có cancel các branch khác, giữ result nào và downstream có thể tiếp tục hay không. Sau khi role tách thành remote Worker, scheduler mới xử lý Lease, task takeover và late result.

### Làm thế nào giới hạn số lần retry và tổng budget?

Budget ít nhất phải tách thành bốn tầng: một model call, một tool call, số iteration của một Agent và Deadline của toàn task hoặc một stage. Ba tầng đầu giới hạn local consumption, tầng ngoài cùng bảo đảm task không bị kéo dài vô hạn vì internal retry.

Retry budget cũng nên chỉ có một owner chính. Nếu Model SDK, role executor và gateway tự retry độc lập, cùng một lỗi sẽ bị khuếch đại thành nhiều vòng nested call. Orchestration layer cần ghi số attempt, Token, time và cost đã tiêu thụ; trước khi retry phải kiểm tra budget còn lại.

### Nên lưu Checkpoint ở đâu?

Checkpoint phù hợp để lưu tại vị trí result đã persist và có thể validate độc lập. Sau khi research stage hoàn thành, trước tiên lưu material list có citation; sau khi data analysis hoàn thành, lưu Artifact có thể reuse; sau khi Reviewer kết thúc, ghi lại lý do pass hoặc reject. Action cần user authorization cũng phải lưu execution parameter trước khi pause.

Session state của một Agent và outer Workflow checkpoint là hai loại data. Loại trước ghi conversation, tool call và internal progress của role; loại sau còn phải lưu current stage, role đã hoàn thành, artifact reference, model và Prompt version, budget usage cùng failure record.

Chỉ ghi chat history vào Redis không tự động giúp tiếp tục cả Workflow từ vị trí “hai research task đã hoàn thành, một data analysis task timeout”. Khi recovery, phải có khả năng dựng lại dependency relationship và confirmed artifact.

Recovery thường cũng không phải tiếp tục từ một dòng Java nào đó. Một role có thể chạy lại từ ReAct node hoặc trước tool call, vì vậy tool dạng read có thể bounded retry; nếu sau này thêm external side effect như transaction và notification, còn phải bảo đảm idempotency.

### Xử lý Agent timeout thế nào?

Concurrency trong process có thể cancel Future chưa hoàn thành sau khi stage Deadline đến, ghi timeout state và giữ artifact đã hoàn thành. Cancel chỉ gửi stop signal cho local task; nếu Agent đã gọi remote model hoặc external tool, còn phải xác nhận remote request đã kết thúc chưa và có tạo side effect nào không.

Sau khi role tách thành service độc lập, caller timeout cũng không chứng minh được Worker cũ đã dừng. Khi scheduler phát hiện Lease hết hạn, trước tiên truy vấn Worker và downstream system xem đã tạo Artifact dùng được hay chưa. Nếu Artifact tồn tại và validation đạt, task có thể tiếp tục trực tiếp; nếu result vẫn unknown, scheduler tăng `attempt` và `fencingToken`, đưa task về `READY`, rồi Worker mới recovery từ Checkpoint gần nhất và Artifact URI.

Sau đó Worker cũ khôi phục và submit result, Fencing Token hoặc version cũ sẽ khiến write này thất bại. Khi task vượt max attempt, chuyển sang `FINAL_FAILED`; hiện trường và artifact đã hoàn thành vẫn được giữ để xử lý thủ công.

Ngoài `lastHeartbeatAt`, còn phải ghi `lastProgressAt` để nhận diện tình trạng “process vẫn sống nhưng ReAct cứ lặp vòng”.

### Khi partial failure, nên wait, degrade hay trả partial result?

Cách xử lý do success criteria của task quyết định và phải được ghi vào task contract trước khi chạy. Khi thiếu dimension bắt buộc, có thể fail toàn bộ; khi nhiều branch cùng loại chỉ cần đạt số lượng quy định, có thể merge theo Quorum; task cho phép hoàn thành một phần thì trả result hiện có và đánh dấu missing item. Backup tool, model và data source là Fallback; task có value hoặc risk cao có thể chuyển sang human để bổ sung.

AgentInvest dùng Best-effort có gate: Researcher phải thành công, Technical Analyst và Sentiment Analyst chỉ cần ít nhất một thành công thì Investment Manager mới tiếp tục. Khi technical analysis lỗi nhưng sentiment result dùng được, Investment Manager có thể tiếp tục dựa trên result hiện có; runtime state vẫn phải giữ technical analysis failure, không được coi lần phân tích này là full success. Khi cả hai role bổ sung đều lỗi, không cho Investment Manager cố ghép conclusion.

![Strategy xử lý partial failure của nhánh Multi-Agent](https://oss.javaguide.cn/github/javaguide/ai/agent/multi-agent-partial-failure-strategies.webp)

### Bù đắp external side effect thế nào?

Refund, gửi email, tạo ticket, sửa code và release version đều thay đổi external state. Có thể giao thống nhất các action này cho Action Executor; Agent phụ trách phân tích và tạo text chỉ submit action intent cùng parameter.

Sau khi nhận request, Action Executor trước tiên validate identity, permission và risk level; high-risk action phải chờ human confirmation. Khi execute, dùng idempotency key và business unique constraint, đồng thời tùy business mà ghi transaction, Outbox hoặc Saga. Sau khi action hoàn thành, tiếp tục verify external state và lưu audit record; khi lỗi, dựa trên step đã execute để chọn compensation hoặc chuyển cho human.

Compensation không đồng nghĩa với database rollback. “Đã gửi một email” thường không thể thực sự thu hồi, chỉ có thể gửi thêm correction notice. Compensation cũng có thể lỗi, nên cần idempotency, retry và human entry point. Có thể xem nguyên lý chi tiết trong [distributed transaction](../../distributed-system/distributed-transaction.md).

## Human-in-the-loop tạm dừng và khôi phục task thế nào?

HITL xảy ra trong quá trình task thực thi. Customer service Agent xác định order có thể refund, nhưng amount vượt approval threshold tự động; lúc đó orchestrator chuyển task sang `WAITING_INPUT`, lưu Checkpoint, căn cứ refund và execution parameter đang chờ, sau đó giải phóng resource hiện tại.

Sau đó approver mở task, có thể approve, reject hoặc sửa parameter. System dùng `runId/taskId` ban đầu để resume, đồng thời validate lại permission của approver, order state, refund amount và task version. Nếu bất kỳ điều kiện nào thay đổi trong thời gian chờ, approval ban đầu phải mất hiệu lực.

Các high-risk action như release code và xóa data cũng có thể dùng cách pause tương tự. Khi Agent thiếu input cần thiết, cần bổ sung tool authorization hoặc conclusion của nhiều role không thể tự merge, cũng có thể chuyển sang waiting state. Framework phụ trách interrupt và recovery mechanism; business code quyết định ai được approve, phạm vi approval và thời hạn hiệu lực.

## Observe một lần chạy Multi-Agent thế nào?

Một lần chạy Multi-Agent sẽ sinh ra nhiều task, Trace cũng nên giữ quan hệ parent-child này:

```text
ResearchRun
├── Planner
│   └── ModelCall
├── SourceResearchStage
│   ├── OfficialDocsResearcher
│   │   └── WebSearchTool
│   └── PaperResearcher
│       └── PaperSearchTool
├── Analyst
│   └── DataAnalysisTool
└── Reviewer
    └── ModelCall
```

Root Span biểu thị toàn bộ `ResearchRun`, ghi `runId`, tenant, task subject cùng code version và Prompt version. Task của mỗi role là một child Span, mang parent task ID, Agent role, model, Skill, state, `attempt` và stop reason. Model và tool call tiếp tục nằm dưới role Span, ghi parameter và result summary, error type, Token, cost và duration.

Có quan hệ parent-child, khi troubleshooting mới nhìn ra thời gian đã tốn ở queue wait, role execution hay final merge, đồng thời xác nhận được tại sao downstream bị skip. Một lần retry phải tiếp tục nằm dưới task của role ban đầu và giữ `attempt` mới, không được giả dạng thành một normal call hoàn toàn mới.

Frontend có thể dùng SSE hoặc WebSocket để hiển thị role start, tool call, result summary và final output, nhưng event stream không nên trở thành source of truth của task state. Recovery sau disconnect, replay và audit vẫn phải đọc persisted Agent state, Workflow run record và Artifact.

Routing result của model cũng phải được ghi vào Trace ngay khi call bắt đầu, ghi provider và model thực tế được sử dụng. Nếu kết thúc execution mới route lại một lần rồi bổ sung log, observation data có thể bị ghi nhầm thành model khác.

OpenTelemetry [GenAI semantic conventions](https://opentelemetry.io/docs/specs/semconv/registry/attributes/gen-ai/) liệt kê các operation name như `invoke_agent`, `invoke_workflow`, `execute_tool` cùng các attribute liên quan đến Agent, Conversation, Tool và Token usage; hiện các convention này vẫn ở trạng thái Development. Tool parameter và result có thể chứa sensitive data, specification cũng nhắc rõ không nên mặc định ghi các field này. Production system nên dùng summary, de-identification, sampling và tiered access.

### Multi-Agent cần những metric riêng nào?

Khi task lỗi, trước tiên xem success rate, timeout rate và retry distribution của từng role, sau đó drill down đến error rate và P95 của model và tool. Một role thường xuyên chạm giới hạn ReAct iteration thường đồng thời làm tăng Token, cost và stage duration.

Parallel stage còn phải ghi queue wait, actual concurrency và duration của branch chậm nhất. Sau khi role bàn giao, tiếp tục thống kê số lần structured result validation thất bại, rework và downstream skip. Cuối cùng xem Token, cost, P95 của toàn task, cũng như thời gian chờ first frontend event và full result lần lượt là bao lâu.

Nhiều branch đã chạy song song nhưng end-to-end duration vẫn do branch chậm nhất và final merge quyết định. Khi parallel không làm nhanh hơn, có thể kiểm tra concurrency limit, external API rate limiting, tool call dài nhất và duration của merge node.

## Đánh giá Multi-Agent thế nào?

Thứ tự tool được ReAct chọn có thể khác nhau mỗi lần, nên khi đánh giá không cần match từng node với một trajectory cố định. Trước tiên kiểm tra Subagent có bàn giao result theo task contract không, sau đó kiểm tra orchestrator có tuân thủ dependency, concurrency và skip condition không, cuối cùng xác nhận merge node có bỏ sót upstream conflict, citation hoặc failure information hay không.

Task nghiên cứu kỹ thuật có thể dùng code để kiểm tra citation format, source time, required field và Artifact có tồn tại không, sau đó đánh giá conclusion có được material hỗ trợ không. Tool registration, role permission, task dependency, concurrency limit, timeout cancellation và context isolation đều có rule xác định, dùng unit test và integration test để verify là đủ.

Recovery capability phải được verify bằng fault injection. Cho một Research Agent timeout, quan sát artifact khác có được giữ lại không và Reviewer có tiếp tục theo success gate không; cho tool trả về invalid data, kiểm tra role có đánh dấu nhầm success không; cung cấp các source xung đột, xác nhận merge node giữ lại disagreement và evidence.

Quality của final recommendation vẫn cần fixed task set, multiple run và human calibration. Định nghĩa của Task, Trial, Grader, Outcome và Transcript xem trong [AI application evaluation system](../llm-basis/llm-evaluation.md).

Multi-Agent còn cần control group. Có thể cho Single Agent dùng toàn bộ tool, sau đó lần lượt chạy fixed Multi-Agent Workflow và Supervisor dynamic orchestration; cả ba group dùng cùng input snapshot, model và budget. Chỉ sau khi so sánh necessary tool call rate, evidence coverage, factual error, P95, Token và cost mới có thể đánh giá việc tách role có thực sự tạo lợi ích hay không.

Role ablation cũng rất trực tiếp. Bỏ một Research Agent, Reviewer hoặc merge node rồi đánh giá lại; nếu quality, coverage và failure recovery gần như không đổi, role đó rất có thể chỉ làm tăng số lần gọi.

## Những lỗi nào thường gặp nhất khi thiết kế Multi-Agent?

### Tên role có thể trực tiếp làm ranh giới Agent không?

Các tên Researcher, Analyst và Reviewer chỉ mô tả phân công role, không chứng minh hệ thống đã tách thành nhiều Agent. Nếu chúng dùng chung một context và một bộ tool, mỗi bước chỉ rewrite text của bước trước, implementation này vẫn gần với Prompt Chain hơn.

Để xác định có thực sự tách được nhiều Agent hay chưa, hãy xem mỗi role có thể độc lập nhận subgoal, hoàn thành task trong context và tool bị giới hạn, đồng thời bàn giao result theo Schema đã quy định hay không. Nếu thiếu những khác biệt này, chỉ đổi một đoạn System Prompt không tạo ra ranh giới Agent rõ ràng.

### Multi-Agent nhất định phải chạy song song không?

Trong AgentInvest, Researcher phải tạo conclusion nền tảng trước, Investment Manager cũng phải đợi upstream task hoàn thành mới có thể merge; chỉ Technical Analysis và Sentiment Analysis không phụ thuộc nhau mới phù hợp chạy song song. Parallel do task dependency quyết định, không có quan hệ tất yếu với số lượng Agent trong system. Reviewer, Handoff và Group Chat thảo luận theo round đều có thể chạy tuần tự. Chưa xác nhận input đã sẵn sàng và result sẽ merge thế nào mà ép concurrent chỉ làm tăng độ phức tạp của state và failure handling.

### Có cần giao deterministic step cho Agent không?

Input có hợp lệ không, role hiện tại có thể gọi tool nào, upstream artifact đã đủ chưa, đều là judgment có rule rõ ràng; giao cho code và state machine xử lý là đủ. Timeout cancellation, retry count và stage order cũng phải do runtime enforce. Agent phù hợp hơn với công việc cần semantic judgment, chẳng hạn chọn hướng truy vấn, giải thích material mâu thuẫn và tổng hợp conclusion của nhiều role. Nhét deterministic rule vào Prompt không chỉ chậm hơn mà còn biến hành vi vốn có thể reproduce ổn định thành vấn đề xác suất.

### Có phải mọi Agent đều nên chia sẻ toàn bộ context không?

Trong nghiên cứu kỹ thuật, Research Agent có thể crawl hàng chục trang và tạo nhiều vòng tool record. Reviewer thường chỉ cần conclusion, citation, missing item và Artifact URI, không cần replay toàn bộ research process. Khi thực sự cần kiểm tra raw material, hãy đọc theo URI. Mỗi Agent chỉ nhận task của mình, tool khả dụng và upstream result cần thiết; cách này vừa kiểm soát Token vừa giảm nhiễu không liên quan đến role hiện tại.

### Có thể để model tự bàn bạc về parallel write conflict không?

Trước hết cần phân biệt conclusion conflict và write conflict. Hai Analysis Agent đưa ra judgment khác nhau trên cùng data có thể giữ evidence riêng, rồi giao Reviewer hoặc merge node phán đoán. Hai branch đồng thời ghi đè `finalOutput` là vấn đề concurrent write, model discussion không thể giải quyết.

Parallel result nên được lưu riêng theo role, final report chỉ cho phép merge node ghi. Các action như gửi notification, tạo ticket và refund còn tạo external side effect, cần tiếp tục dùng idempotency key, unique constraint và transaction để bảo vệ; không thể trông chờ Prompt tránh việc execute trùng.

### Chỉ thiết kế normal execution path sẽ có vấn đề gì?

Giả sử hai Research Agent chạy song song, một thành công và một timeout. Lúc này giữ material hiện có để đưa cho Reviewer hay xác định tài liệu không đủ rồi kết thúc task phụ thuộc vào success gate đã thỏa thuận trước. Khi thiết kế task phải bao phủ các state như partial success, timeout, cancel, thiếu artifact bắt buộc và downstream skip. Nếu không, normal path trên flowchart tuy chạy được, nhưng khi gặp exception đầu tiên trong production, orchestrator lại không biết nên tiếp tục, degrade hay stop.

### Vì sao không thể chỉ kiểm tra final response?

Ngay cả khi một tool call truy vấn thất bại, model vẫn có thể tạo report trôi chảy và đầy đủ cấu trúc. Chỉ nhìn final text rất dễ coi kết quả này là success.

Khi acceptance test còn phải kiểm tra tool bắt buộc có call thành công không, mỗi role có bàn giao theo contract không, citation có truy ngược được về raw material không và merge node có che giấu failure branch không. Final response chỉ là một loại artifact, không thể thay thế việc acceptance test execution process.
