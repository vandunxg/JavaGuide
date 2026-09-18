---
title: "AI Observability và Trace: Cách khôi phục toàn bộ quá trình thực thi của Agent"
description: Bắt đầu từ một request có interface trả về thành công nhưng câu trả lời sai, giải thích thiết kế Trace, cách tách Span, truyền context, sampling, xử lý dữ liệu nhạy cảm và cách triển khai trong dự án Spring AI.
category: Phát triển ứng dụng AI
tag:
  - AI Observability
  - OpenTelemetry
  - AI Agent
  - Spring AI
  - Thiết kế hệ thống AI
head:
  - - meta
    - name: keywords
      content: AI Observability,AI Observability,Agent Trace,OpenTelemetry,Spring AI,LLM Observability,RAG Observability,LLM Observability
---

Interface trả về 200, thời gian xử lý cũng bình thường, nhưng quy định hoàn tiền mà chatbot chăm sóc khách hàng đưa ra lại sai.

Mở dashboard giám sát, bạn chỉ thấy request này đã gọi model và tạo câu trả lời bình thường. Điều tra sâu hơn thì khó rồi: knowledge base không recall đúng tài liệu, Prompt bị ghép sai, model bỏ qua context, hay một tool nào đó trả về dữ liệu cũ? Log của interface thông thường không trả lời được những câu hỏi này.

Một request AI có thể đi qua rewrite câu hỏi, truy vấn knowledge base, rerank, gọi model, gọi tool và kiểm tra kết quả. Agent còn có thể lặp lại một vài bước trong đó. Mỗi bước có vẻ đều gọi thành công, nhưng kết quả sai vẫn được truyền sang bước tiếp theo, cuối cùng tạo ra một câu trả lời có format bình thường nhưng nội dung có vấn đề.

Khi điều tra, cần lần ngược theo quá trình thực thi này: lúc đó đã dùng model và Prompt nào, đã truy vấn được những document nào, đã gọi tool gì, Agent retry mấy lần, cuối cùng câu trả lời tham chiếu những kết quả nào.

Chỉ khi những thông tin này nằm trong cùng một call chain, bạn mới có thể tiếp tục định vị vấn đề.

## Vì sao ứng dụng AI không thể chỉ xem interface có thành công hay không?

Monitoring backend truyền thống chủ yếu xem QPS, error rate, P99 latency, CPU, memory và slow query của database. Những chỉ số này phản ánh service có bất thường hay không, nhưng không giải thích được vì sao model tạo ra câu trả lời hiện tại.

Lấy RAG làm ví dụ, nếu rewrite câu hỏi bị lệch thì kết quả recall phía sau thường cũng lệch theo; document đúng dù được recall vẫn có thể bị loại khỏi `topK` khi rerank; thứ tự document không có vấn đề nhưng vẫn có thể bị cắt vì Prompt quá dài. Toàn bộ quá trình không throw exception, interface vẫn trả về 200.

Trong scenario Agent còn có thêm vấn đề về tool và trạng thái thực thi. Interface của tool gọi thành công, nhưng dữ liệu nghiệp vụ nhận được có thể đã hết hạn; một lần retry sau timeout có thể lặp lại thao tác có side effect; khi tổng hợp nhiều Agent, cũng có thể bỏ sót một kết quả đã hoàn thành.

Muốn điều tra đến bước cụ thể, Trace ít nhất phải bao phủ model, truy vấn, tool, Agent và kết quả cuối cùng. Trace thất bại đã xác nhận còn có thể được tích lũy thành Badcase để dùng cho đánh giá hồi quy sau này.

Trace có thể khôi phục các bước hệ thống thực sự đã thực thi, nhưng không đọc được cơ chế suy luận thực bên trong model. Phần text tư duy model trả về cũng chỉ là một output, không thể trực tiếp xem là lời giải thích đầy đủ cho quyết định.

## Metrics, Logs, Trace, Evaluation và Audit khác nhau thế nào?

Trong AI Observability, Metrics, Logs, Trace, Evaluation và Audit thường xuất hiện cùng nhau.

Mỗi khái niệm trong số này giải quyết một loại vấn đề khác nhau.

| Tín hiệu   | Câu hỏi chính cần trả lời                  | Nội dung điển hình                                                                |
| ---------- | ------------------------------------------ | --------------------------------------------------------------------------------- |
| Metrics    | Toàn bộ hệ thống có bất thường không?      | Lưu lượng request, error rate, P95 latency, Token usage, error rate của tool      |
| Logs       | Đã xảy ra event gì tại một thời điểm?      | Stack exception, thay đổi trạng thái, nguyên nhân retry, cảnh báo nghiệp vụ       |
| Trace      | Một request đã trải qua những gì?          | Gọi model, truy vấn, gọi tool, thực thi Agent và quan hệ giữa upstream/downstream |
| Evaluation | Chất lượng output có đạt không?            | Tính đúng, độ trung thành, lựa chọn tool, mức độ hoàn thành task, security        |
| Audit      | Ai đã thực hiện thao tác gì với quyền nào? | Danh tính user, lịch sử phê duyệt, quyết định quyền, thao tác ghi ra bên ngoài    |

![Các vấn đề mà từng tín hiệu Observability giải quyết](https://oss.javaguide.cn/github/javaguide/ai/llm/ai-observability-signals-overview.webp)

Mối quan hệ phối hợp của chúng có thể giải thích bằng một câu: **Metrics phát hiện bất thường, Trace định vị quá trình, Evaluation đánh giá chất lượng, Audit truy vết trách nhiệm.**

Ví dụ, khi Token usage đột ngột tăng, Metrics có thể nhanh chóng kích hoạt cảnh báo. Lần theo Trace có mức tiêu thụ cao trong khoảng thời gian bất thường, bạn có thể thấy một Agent đã liên tục gọi model 12 lần. Còn 12 lần gọi đó có cần thiết hay không, câu trả lời cuối cùng có đúng hay không, vẫn phải giao cho rule Evaluation phán đoán.

Logs cũng không thể bị Trace thay thế hoàn toàn. Trace phù hợp để hiển thị quan hệ gọi và thời gian xử lý, còn log phù hợp hơn để ghi các event rời rạc và exception chi tiết. Cách làm thực tế là để log tự động kèm `traceId` và `spanId`, nhờ đó có thể từ một Trace chuyển đến log liên quan, hoặc từ error log truy ngược toàn bộ call chain.

## Phân biệt Trace, Span, Session, Run và Attempt thế nào?

Năm khái niệm này rất dễ bị trộn lẫn. OpenTelemetry định nghĩa rõ Trace và Span, còn Session, Run, Attempt thường do ứng dụng tự định nghĩa.

| Khái niệm | Ý nghĩa                      | Ví dụ                                                       |
| --------- | ---------------------------- | ----------------------------------------------------------- |
| Session   | Một phiên hội thoại liên tục | 8 lượt hội thoại giữa user và customer-service Agent        |
| Run       | Một lần thực thi task        | User yêu cầu “giúp tôi đăng ký hoàn tiền” ở lượt thứ 5      |
| Trace     | Một call chain end-to-end    | Từ lúc API nhận request đến lúc trả về kết quả xử lý        |
| Span      | Một thao tác trong Trace     | Truy vấn knowledge base, gọi model, thực thi tool hoàn tiền |
| Attempt   | Một lần thử của một bước     | Lần retry thứ 2 sau khi gọi model timeout                   |

Với request đồng bộ, việc này khá dễ hiểu: tạo Trace khi task bắt đầu, kết thúc khi trả response, một Run thường vừa đúng nằm trong một Trace.

Khi chuyển sang task dài như phê duyệt thủ công hoặc khôi phục bất đồng bộ, mối quan hệ này không còn đúng. Ví dụ, sau khi Agent gửi đơn, có thể phải chờ vài giờ mới nhận được kết quả phê duyệt, không cần để Span ban đầu mở mãi. Khi chuyển sang trạng thái chờ thì kết thúc Trace hiện tại, đồng thời lưu `runId`; sau khi được phê duyệt thì tạo một Trace mới và tiếp tục sử dụng `runId` cũ. Cách này vừa liên kết được hai đoạn thực thi trước sau, vừa không tạo một Trace kéo dài vài giờ.

Phạm vi của Session còn lớn hơn một tầng. Nó nối nhiều lượt hội thoại, mỗi lượt hội thoại hoặc mỗi lần thực thi task đều có thể tạo Trace riêng. Mỗi ID quản lý một tầng:

```text
sessionId: nối cùng một phiên hội thoại nhiều lượt
runId: nối một task, bao gồm cả thực thi sau khi tạm dừng và khôi phục
traceId: định danh call chain do một lần thực thi thực tế tạo ra
spanId: định danh một thao tác trong call chain
attempt: định danh lần thử thứ mấy của cùng một bước
```

![Quan hệ phân cấp giữa Session, Run, Trace, Span và Attempt](https://oss.javaguide.cn/github/javaguide/ai/llm/ai-observability-id-hierarchy.webp)

Retry cũng phải được xử lý riêng. Mỗi Attempt tạo một Span mới và dùng `attempt` để đánh dấu số lần. Nếu không, lần timeout đầu tiên và lần thành công thứ hai sẽ trộn vào cùng một record, về sau rất khó khôi phục quá trình thực thi thực tế.

## Nên tách Span cho một request Agent thế nào?

Granularity khi tách Span quyết định Trace có thực sự hữu ích hay không. Nếu tách quá thô, chỉ thấy “Agent thực thi 8 giây”; nếu tách quá nhỏ, mỗi method thông thường đều là một Span, call chain sẽ đầy các node không giúp ích cho việc điều tra.

Lấy một Agent có thể truy vấn order và đăng ký hoàn tiền làm ví dụ, một Trace có thể tách như sau:

```text
agent.run
├── ai.intent.classify
│   └── gen_ai.client.operation
├── ai.rag.retrieve
│   ├── gen_ai.client.operation        # Tạo query vector
│   ├── db.vector.client.operation     # Truy vấn vector
│   └── ai.rag.rerank
├── invoke_agent                       # Thực thi customer-service Agent
│   ├── gen_ai.client.operation        # Lần gọi model đầu tiên
│   ├── execute_tool                   # Truy vấn order
│   │   └── HTTP GET /orders/{id}
│   ├── gen_ai.client.operation        # Tiếp tục phán đoán dựa trên kết quả tool
│   └── execute_tool                   # Tạo yêu cầu hoàn tiền
│       └── POST /refunds
└── ai.output.validate
```

![Sơ đồ tách Span của một request Agent](https://oss.javaguide.cn/github/javaguide/ai/llm/ai-observability-agent-span-trace.webp)

Ở đây không tạo Span riêng cho các method thông thường như render Prompt template hay nối string. Thông thường, chỉ cần thỏa mãn một trong các điều kiện dưới đây thì mới đáng được ghi lại độc lập:

- Gọi hệ thống bên ngoài, chẳng hạn model service, vector database và business API;
- Bản thân là một giai đoạn nghiệp vụ quan trọng, chẳng hạn truy vấn, rerank, planning và kiểm tra kết quả;
- Có timeout, retry hoặc degradation strategy độc lập;
- Cần thống kê riêng latency, cost, error rate hoặc quality;
- Sau khi xảy ra vấn đề, developer thực sự cần xác định nó đã thực thi hay chưa, đã thực thi bao nhiêu lần.

Một Span tối thiểu phải trả lời được bốn câu hỏi: đã thực thi gì, bắt đầu và kết thúc lúc nào, kết quả thành công hay không, quan hệ của nó với các bước khác là gì.

### Nên thiết kế quan hệ cha con thế nào?

Lời gọi đồng bộ thường sử dụng Span cha con trực tiếp. Span cha biểu thị một giai đoạn lớn hơn, Span con biểu thị thao tác bên trong giai đoạn đó.

Khi Agent gọi song song nhiều chuyên gia, Span của mỗi chuyên gia đều có thể gắn dưới cùng một Span orchestration. Khoảng thời gian của chúng sẽ chồng lấn, trang Trace tự nhiên sẽ hiển thị được quan hệ song song.

```text
invoke_workflow
├── invoke_agent: technical-analysis   ──────────┐
├── invoke_agent: sentiment-analysis   ───────┐  │ song song
└── invoke_agent: portfolio-manager            └── tổng hợp
```

Một Span chỉ có thể có một Span cha. Nếu một task aggregation được kích hoạt bởi nhiều message độc lập, có thể chọn consumer operation hiện tại làm Span cha, rồi dùng Span Link liên kết các context upstream khác. Đừng tự ghép một cây không phù hợp với mô hình Trace chỉ để thể hiện “nhiều node cha”.

### Nên đặt tên Span thế nào?

Tên Span phải ổn định, cardinality thấp, tránh đưa user question, order number hoặc URL đầy đủ vào.

Khuyến nghị:

```text
invoke_agent customer-service
execute_tool query_order
ai.rag.retrieve
POST /refunds
```

Không khuyến nghị:

```text
Trả lời câu hỏi của user: Vì sao order 202608160001 của tôi vẫn chưa được hoàn tiền
Truy vấn order 202608160001
```

Tên ở ví dụ sau sẽ khiến số lượng tên Span tăng liên tục theo input của user, vừa khó aggregate, vừa làm tăng rõ rệt chi phí storage và index. Order number có thể được lưu dưới dạng attribute được kiểm soát, hoặc chỉ lưu hash; không cần đưa vào tên Span.

### OpenTelemetry đã thống nhất Agent Span chưa?

OpenTelemetry đã cung cấp semantic conventions cho GenAI, bao phủ model, Agent và một phần thao tác workflow. Tài liệu Agent Span hiện bao gồm các operation name như `invoke_agent`, `invoke_workflow`, `plan`, `execute_tool`.

Tuy nhiên, tại thời điểm viết bài, [semantic conventions GenAI của OpenTelemetry](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/README.md) và [conventions Agent Span](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-agent-spans.md) vẫn được đánh dấu là `Development`. Điều này có nghĩa là attribute và naming vẫn có khả năng được điều chỉnh.

Về mặt engineering, có thể ưu tiên tái sử dụng các field chung đã được định nghĩa, rồi thêm namespace riêng cho các giai đoạn nghiệp vụ, chẳng hạn `ai.rag.*`, `ai.guardrail.*`. Đồng thời ghi version của convention vào thư viện instrumentation, tập trung adapt khi upgrade, không rải các field riêng của vendor vào code nghiệp vụ rồi xem chúng như standard ổn định.

## Một Trace nên ghi lại những thông tin nào?

Trace ghi càng nhiều thì khi điều tra vấn đề càng thuận tiện, nhưng storage cost và rủi ro rò rỉ cũng tăng theo. Trên production có thể ghi metadata có cấu trúc trước, sau đó quyết định có lưu content hay không dựa trên environment và quyền hạn.

### Thông tin request và version

Một request AI tối thiểu phải có thể xác định code và config đã chạy tại thời điểm đó:

- `service.name`, environment và service version;
- `sessionId`, `runId`, tenant identifier;
- version của Agent, Workflow, Prompt và knowledge base;
- nhóm canary, nhóm experiment và feature flag;
- entry point của request, caller và region.

Chỉ ghi content của Prompt template là chưa đủ. Template không đổi, nhưng system Prompt, Few-shot example, context truy vấn hoặc model parameter thay đổi thì output vẫn có thể hoàn toàn khác. Vì vậy, tốt nhất lưu thêm version hoặc hash của config thực sự có hiệu lực.

### Thông tin gọi model

Model Span thường cần ghi lại:

- model provider, model được request và model thực tế response;
- temperature, Token output tối đa, có trả về dạng stream hay không;
- input Token, output Token, Token cache hit;
- latency của Token đầu tiên và tổng thời gian xử lý;
- Finish Reason, rate limit, timeout và số lần retry;
- estimated cost hoặc billing correlation ID.

Model alias có thể trỏ đến các version khác nhau. Chỉ ghi `model=smart-model` trong business config sẽ khiến về sau rất khó xác nhận thực tế đã chạy model nào. Nếu response của vendor cung cấp model version hoặc response ID thì nên giữ lại cùng.

### Thông tin truy vấn RAG

Các field có giá trị tương đối cao trong giai đoạn truy vấn gồm:

- version của knowledge base và index;
- hash hoặc content được kiểm soát của câu hỏi gốc và câu hỏi đã rewrite;
- retrieval strategy, điều kiện filter, `topK` và similarity threshold;
- document ID được recall, score, thứ tự và data version;
- model rerank, thứ tự trước và sau rerank;
- document ID cuối cùng được đưa vào Prompt;
- query latency, số lượng kết quả trả về và cờ empty recall.

Nếu chỉ ghi “truy vấn vector database thành công”, sau khi xuất hiện câu trả lời sai vẫn không thể xác định tài liệu đúng có được recall hay không. Tối thiểu phải lưu document identifier, version và score. Có đưa toàn văn document vào Trace hay không cần được quyết định riêng theo data classification.

### Thông tin gọi tool

Tool là entry point để Agent kết nối với hệ thống nghiệp vụ thực tế, nên ghi lại:

- tên tool, version và call ID;
- parameter summary hoặc parameter hash;
- kết quả kiểm tra quyền và phê duyệt thủ công;
- timeout, retry, idempotency key và execution status;
- result summary, business error code và side-effect marker;
- Trace context của downstream service.

“Tool call thành công” không nhất thiết có nghĩa là nghiệp vụ thành công. Response body của HTTP 200 có thể là `stock=0`, interface hoàn tiền cũng có thể trả về “yêu cầu trùng lặp”. Span status, HTTP status và business result nên được ghi riêng.

### Thông tin Agent và workflow

Span của Agent hoặc workflow còn cần bổ sung:

- role, version của Agent và tập tool được phép sử dụng;
- bước hiện tại, số vòng lặp và số lần iteration tối đa;
- routing result, stop reason và degradation reason;
- các nhánh song song đã hoàn thành, timeout hay bị cancel;
- khi tổng hợp thực tế đã sử dụng những kết quả upstream nào;
- kết quả validation có cấu trúc của output cuối cùng.

Các field này phải trả lời một câu hỏi rất cụ thể: rốt cuộc câu trả lời được tạo ra dựa trên những thông tin nào. Đặc biệt trong hệ thống nhiều Agent, “một chuyên gia đã hoàn thành” và “Agent tổng hợp đã sử dụng kết quả của nó” là hai việc khác nhau, tốt nhất nên ghi riêng.

## Phân biệt field cardinality thấp và cardinality cao thế nào?

Hệ thống Observability cần aggregate và index dữ liệu. Thiết kế field không phù hợp sẽ khiến time series của metric và Trace index phình to nhanh chóng.

Giá trị của field cardinality thấp có phạm vi hữu hạn, phù hợp để đưa vào metric và Trace:

```text
model.provider = openai
model.name = gpt-x
agent.role = customer-service
tool.name = query_order
result.status = success
environment = production
```

Giá trị của field cardinality cao rất nhiều, thường chỉ phù hợp để đưa vào Trace, một số field còn cần tránh index:

```text
user.id
session.id
document.id
model.response.id
tool.call.id
```

Prompt, Completion, tool parameter và document content không chỉ có cardinality cao, mà còn có thể rất dài và chứa thông tin nhạy cảm; chúng phù hợp hơn khi được xem là event được kiểm soát, lưu trữ content độc lập, hoặc chỉ lưu hash và object address.

Thiết kế Observability của Spring AI cũng áp dụng nguyên tắc tương tự: [key cardinality thấp đi vào Metrics và Trace, key cardinality cao chỉ đi vào Trace](https://docs.spring.io/spring-ai/reference/observability/index.html). Khi tự bổ sung instrumentation nghiệp vụ, chúng ta cũng nên giữ ranh giới này.

## Ứng dụng AI nên tập trung monitoring những metric nào?

Metric nên được thiết kế xoay quanh user experience, stability, cost và quality. Các metric dưới đây khá hữu ích trong phần lớn ứng dụng AI.

### Metric tầng request

- Lưu lượng request, success rate và số lượng concurrent;
- P50, P95, P99 latency end-to-end;
- timeout rate, cancellation rate và degradation rate;
- latency của Token đầu tiên và thời gian response đầy đủ;
- phân bố traffic theo entry point, tenant và version.

Interface streaming cần phân biệt latency của Token đầu tiên với tổng thời gian xử lý. User thường quan tâm mất bao lâu để thấy nội dung đầu tiên, còn task batch quan tâm hơn đến lúc toàn bộ task kết thúc.

### Metric tầng model

- Số lần gọi model, error rate và rate limit;
- input, output và tổng Token;
- phân bố Token của từng request;
- Token cache hit;
- số lần retry và fallback model;
- cost thống kê theo model, scenario và tenant.

Token usage trung bình rất dễ che lấp long tail, tối thiểu còn phải theo dõi P95 và giá trị lớn nhất. Khi một Agent thỉnh thoảng rơi vào vòng lặp, giá trị trung bình chưa chắc thay đổi rõ rệt, còn percentile cao sẽ sớm phơi bày vấn đề.

### Metric tầng RAG

- Empty recall rate và số lượng document có hiệu lực;
- thời gian truy vấn, rerank và dựng context;
- phân bố similarity score;
- tỷ lệ cắt document và context Token;
- coverage của citation, faithfulness và các metric Evaluation offline hoặc online khác.

Similarity cao không có nghĩa là câu trả lời chắc chắn đúng, chỉ cho biết mức độ liên quan trong vector space. Chất lượng truy vấn cuối cùng vẫn phải kết hợp data đã gán nhãn và Evaluation answer.

### Metric tầng Agent và tool

- Số lần gọi model, số lần gọi tool và số lần iteration của mỗi Run;
- success rate, business failure rate và timeout rate của tool;
- tool call không hợp lệ, call trùng lặp và call bị từ chối;
- tỷ lệ human takeover, thời gian chờ phê duyệt;
- task completion rate, partial success rate và phân bố nguyên nhân thất bại;
- thời gian xử lý của nhánh nhiều Agent, kết quả bị thiếu và tỷ lệ degradation khi tổng hợp.

Không thể trực tiếp sao chép threshold metric từ project khác. Một câu hỏi đơn giản gọi model 6 lần có thể là bất thường rõ rệt, trong khi task nghiên cứu kỹ thuật phức tạp gọi model 6 lần lại hoàn toàn bình thường. Trước tiên hãy xây baseline theo scenario và version, sau đó đặt cảnh báo cho các thay đổi lệch baseline; cách này đáng tin cậy hơn nhiều so với việc cấu hình cùng một threshold cố định cho mọi Agent.

## Dùng Trace để định vị AI Badcase thường gặp thế nào?

Một Trace hữu ích phải hỗ trợ điều tra từng lớp theo đúng thứ tự thực thi thực tế. Dưới đây dùng ba vấn đề thường gặp để minh họa.

### RAG trả về câu trả lời sai, nên xem đâu trước?

Có thể kiểm tra theo thứ tự sau:

1. Xem rewrite question Span, xác nhận query truy vấn có lệch ý định ban đầu của user hay không;
2. Xem retrieval Span, xác nhận document đúng có được recall hay không;
3. Xem kết quả rerank, xác nhận document đúng có bị xếp ra ngoài `topK` hay không;
4. Xem record dựng context, xác nhận document có bị cắt vì giới hạn Token hay không;
5. Xem model Span cuối cùng, xác nhận đã gửi version Prompt nào và những document ID nào;
6. Cuối cùng kết hợp kiểm tra citation và Evaluation faithfulness để xác định model có trả lời tách khỏi tài liệu hay không.

Call chain này có thể tách “câu trả lời sai” thành vấn đề recall, vấn đề ranking, vấn đề context và vấn đề generation. Việc sửa chữa sau đó cũng sẽ có mục tiêu cụ thể hơn.

### Vì sao refund tool thực thi hai lần?

Trước tiên xem số lượng `execute_tool` Span và `attempt`. Nếu lần gọi đầu tiên timeout, Agent phát động lần gọi thứ hai, cần tiếp tục kiểm tra:

- Hai request có sử dụng cùng một idempotency key ổn định hay không;
- Sau timeout lần đầu, downstream có vẫn hoàn thành thao tác hoàn tiền hay không;
- Retry strategy đến từ model, Agent framework hay HTTP client;
- Sau khi kết quả tool bị mất, Agent có hiểu nhầm “unknown” thành “failure” hay không;
- Business record và audit log của downstream có xác nhận hai side effect đã xảy ra hay không.

Trace có thể chứng minh application đã phát động những call nào, nhưng không thể thay thế business record của hệ thống hoàn tiền. Nếu call timeout, ngay cả khi Span status là `ERROR`, hoặc application attribute ghi task là `CANCELED`, cũng không thể dựa vào đó để kết luận thao tác phía xa đã dừng.

### Vì sao tổng hợp nhiều Agent lại bỏ sót một kết quả?

Trước tiên tìm workflow Span, sau đó kiểm tra status và khoảng thời gian của từng Agent con:

- Agent con có thực sự khởi động hay không;
- Là success, failure, timeout hay bị cancel;
- Kết quả có được ghi trả qua message hoặc shared state hay không;
- Khi Agent tổng hợp khởi động, kết quả phụ thuộc đã sẵn sàng hay chưa;
- Trong input tổng hợp thực tế có result ID của những Agent nào;
- Khi thiếu kết quả, hệ thống đi vào nhánh failure, wait hay degradation.

Nhiều cuộc điều tra bỏ sót “input tổng hợp”. Chỉ có Span hoàn thành của Agent con mà không có danh sách input của giai đoạn aggregation thì vẫn không thể xác định kết quả bị mất khi truyền tải, hay Agent tổng hợp vốn không sử dụng nó.

## Làm thế nào để giữ Trace qua nhiều Agent, task bất đồng bộ và message queue?

Trong code đồng bộ một thread, Trace context thường có thể tự động truyền theo call stack. Khi chuyển thread, process hoặc message queue, cần xử lý rõ việc propagation.

![Cách truyền Trace context qua ranh giới thread, process và message](https://oss.javaguide.cn/github/javaguide/ai/llm/ai-observability-context-propagation.webp)

### Truyền qua service thế nào?

[W3C Trace Context](https://www.w3.org/TR/trace-context/) định nghĩa `traceparent` và `tracestate` request header dùng chung. `traceparent` dùng để truyền Trace ID, parent Span ID và sampling flag, còn `tracestate` có thể mang thêm thông tin mở rộng liên quan đến vendor.

Khi gọi downstream service qua HTTP, RPC hoặc message queue, bên gửi phải inject context, bên nhận phải extract context rồi tạo child Span. Message queue thường đưa các field này vào message Header, không phải message body nghiệp vụ.

`traceparent` và `tracestate` cũng không nên chứa user identity, order number hay business data khác. Specification W3C yêu cầu rõ phải tránh đưa thông tin nhận diện cá nhân vào các field này.

### Vì sao chuyển thread lại làm đứt chain?

Context hiện tại của OpenTelemetry Java mặc định được lưu dựa trên `ThreadLocal`. Sau khi task được submit vào thread pool thông thường, nếu không có auto-instrumentation hoặc wrapper tường minh thì child thread không nhìn thấy context của thread gọi.

Khi sử dụng OpenTelemetry API, có thể capture context trước khi submit task:

```java
Context context = Context.current();

executorService.submit(context.wrap(() -> {
    supplementalAgent.run(task);
}));
```

`Runnable`, `Callable` và `ExecutorService` đều có cách wrapper context tương ứng, có thể tham khảo [OpenTelemetry Java API](https://opentelemetry.io/docs/languages/java/api/) để biết chi tiết.

Nếu project sử dụng Spring Boot, ưu tiên dùng khả năng tự config của framework cho HTTP client, TaskExecutor và Micrometer Context Propagation. Khi tự `new` client hoặc thread pool, propagation tự động thường không thể có hiệu lực.

Virtual thread cũng không thay đổi yêu cầu cơ bản về propagation của Trace context. Nó giải quyết vấn đề cost của thread và concurrency blocking, không thay chúng ta định nghĩa quan hệ nhân quả giữa các task. Có cần propagation thủ công hay không vẫn phụ thuộc vào thư viện instrumentation, executor và cơ chế context được sử dụng.

### Lời gọi streaming có chắc chắn tự động liên kết không?

Không thể mặc định như vậy.

Lấy tài liệu hiện tại của Spring AI 2.0.0 làm ví dụ, lời gọi đồng bộ OpenAI và Anthropic có thể gắn HTTP Span chính xác dưới model Span; path bất đồng bộ của lời gọi streaming sẽ chuyển sang `ForkJoinPool.commonPool()`, khiến HTTP Span được ghi lại nhưng không trở thành child node của model Span. Đây là giới hạn hiện tại được ghi rõ trong [tài liệu Observability chính thức của Spring AI](https://docs.spring.io/spring-ai/reference/observability/index.html#_chat_model).

Ví dụ này cho thấy sau khi framework cung cấp auto-instrumentation, vẫn phải dùng integration test để kiểm tra topology thực tế. Tối thiểu cần bao phủ lời gọi đồng bộ, lời gọi streaming, thread pool, virtual thread, message queue và tool call cross-service, xác nhận `traceId` không bị mất ở boundary.

### Liên kết một task qua nhiều Trace thế nào?

Task Agent bất đồng bộ có thể chạy rất lâu, giữa chừng còn chờ input thủ công. Không cần giữ một Span mở mãi.

Có thể xử lý như sau:

- Dùng `runId` để liên kết toàn bộ business task;
- Tạo một Trace mới cho mỗi lần thực thi thực tế;
- Lưu Trace ID trước đó trong task state;
- Khi khôi phục thực thi, liên kết Trace trước qua attribute hoặc Span Link;
- Dùng event time và state version để khôi phục thứ tự task.

Cách này vừa giữ được tính liên tục của nghiệp vụ, vừa tránh Span quá dài gây áp lực cho sampling và backend storage.

## Có thể lưu Prompt và tool result đầy đủ trong Trace không?

Xét từ góc độ điều tra vấn đề, lưu đầy đủ content là tiện nhất. Nhưng xét từ góc độ security và compliance, đây thường cũng là cách có rủi ro cao nhất.

Prompt, model answer và tool result có thể chứa:

- Tên, điện thoại, địa chỉ và thông tin giấy tờ của user;
- Code, contract và dữ liệu tài chính nội bộ của doanh nghiệp;
- System Prompt, tool definition và security policy;
- Access Token, Cookie và thông tin kết nối database;
- Content trong knowledge base bên thứ ba được bảo vệ bởi copyright hoặc quyền hạn.

Vì vậy, mặc định trên production nên ghi metadata và summary, còn content field chỉ bật khi cần. Có thể xử lý theo các tầng sau:

1. **Tầng mặc định**: lưu version, Token, latency, status, document ID, parameter hash và các thông tin có cấu trúc khác.
2. **Tầng masking**: lưu Prompt và result đã được masking theo rule đối với phone number, email, Token và các thông tin khác.
3. **Tầng debug giới hạn**: chỉ tạm thời bật content đầy đủ cho environment, tenant hoặc Trace được chỉ định, đồng thời đặt retention period ngắn hơn.
4. **Tầng audit**: ghi lại ai yêu cầu, ai phê duyệt, ai đã đọc content nhạy cảm trong Trace.

Spring AI mặc định không export Prompt, Completion, tool parameter, tool result và vector query result, chính vì các content này có thể lớn và chứa thông tin nhạy cảm. Các content attribute như `gen_ai.input.messages`, `gen_ai.output.messages`, `gen_ai.system_instructions` trong OpenTelemetry GenAI cũng thuộc nhóm field cần chủ động bật.

Ngoài masking, còn phải xử lý đồng thời access control, encryption khi truyền và lưu trữ, retention period, tenant isolation và cơ chế xóa. [Hướng dẫn Observability cho hệ thống GenAI và Agent của Microsoft](https://learn.microsoft.com/en-us/security/zero-trust/sfi/observability-ai-systems) cũng nhấn mạnh phạm vi thu thập và lưu giữ cần được thống nhất rõ giữa nhu cầu forensics, data minimization, yêu cầu lưu trú dữ liệu, compliance và quyền hạn.

Một số dữ liệu hoàn toàn không nên đi vào Trace, chẳng hạn plain-text password, long-lived key và authentication Token đầy đủ. Ngay cả khi đặt “chỉ administrator có thể xem”, cũng không đáng để chịu rủi ro thêm.

## Nên thiết kế Trace sampling thế nào?

Trong môi trường test có thể tạm thời bật full Trace, nhưng ghi full lâu dài trên production thường rất tốn kém. Số lượng Span, kích thước attribute và thời gian thực thi của request AI đều có thể cao hơn rõ rệt so với interface thông thường, nên sampling strategy cần được thiết kế riêng.

### Head sampling và tail sampling khác nhau thế nào?

Head sampling quyết định ngay khi request vừa vào hệ thống. Ví dụ, giữ lại ngẫu nhiên 10% theo Trace ID. Ưu điểm là triển khai đơn giản, cost có thể kiểm soát; nhược điểm là lúc quyết định chưa biết request cuối cùng có failure hoặc đặc biệt chậm hay không.

Tail sampling chờ thêm nhiều Span của một Trace đến rồi mới quyết định. Nó có thể ưu tiên giữ lại:

- Trace xuất hiện error hoặc timeout;
- request chậm vượt threshold của scenario;
- request có Token hoặc cost cao bất thường;
- request xảy ra degradation, fallback và human takeover;
- request chạm security policy hoặc thực thi tool rủi ro cao;
- request của version mới và canary experiment.

Tail sampling cần Collector tạm lưu Trace, làm tăng cost memory và waiting, đồng thời phải xem xét cách Collector phân tán đưa cùng một Trace đến cùng một decision node. [Tail Sampling Processor](https://explorer.opentelemetry.io/collector/components/contrib-tailsamplingprocessor?type=processor) của OpenTelemetry Collector hiện đánh dấu Trace là Beta; trước khi dùng trên production cần stress test capacity và behavior khi discard.

![Sự khác nhau giữa head sampling và tail sampling của Trace](https://oss.javaguide.cn/github/javaguide/ai/llm/ai-observability-head-tail-sampling.webp)

### Combination strategy thực tế là gì?

Production có thể sử dụng strategy phân tầng:

```text
Request thành công thông thường: sampling tỷ lệ thấp
Request error, timeout và latency cao: cố gắng giữ lại
Token cao, cost cao và số vòng lặp bất thường: cố gắng giữ lại
Write tool rủi ro cao, phê duyệt thủ công và security event: đưa vào audit chain độc lập
Version canary và tenant trọng điểm: tăng tỷ lệ trong thời gian được kiểm soát
```

Cũng cần bảo đảm quyết định sampling nhất quán trong phạm vi một Trace. Nếu root Span được giữ lại nhưng child Span quan trọng lại bị loại riêng, cuối cùng chỉ nhận được một call chain không đầy đủ.

Dữ liệu audit không nên hoàn toàn phụ thuộc vào Trace sampling. Với thao tác rủi ro cao như hoàn tiền, chuyển khoản, xóa resource, ngay cả khi business Trace không được sample thì audit record vẫn nên được lưu độc lập theo yêu cầu nghiệp vụ và compliance.

## Đưa Trace production trở lại hệ thống Evaluation thế nào?

Giá trị của Trace không nên dừng ở việc “chỉ tra cứu khi có vấn đề”. Production Badcase có thể liên tục được đưa trở lại theo quy trình sau:

```text
Cảnh báo production hoặc feedback của user
        ↓
Định vị Trace bất thường
        ↓
Xác nhận root cause và bổ sung human annotation
        ↓
Sau masking, thêm vào tập Badcase
        ↓
Sửa Prompt, RAG, tool hoặc logic orchestration
        ↓
Replay offline và Evaluation hồi quy
        ↓
Release canary và tiếp tục quan sát
```

Khi replay cần phân biệt hai mục tiêu.

Mục tiêu thứ nhất là **tái hiện vấn đề khi đó**. Lần replay này phải cố định model version, Prompt, document snapshot, tool response, random parameter và timeout config, cố gắng khôi phục điều kiện ban đầu.

Mục tiêu thứ hai là **xác minh hiệu quả của version mới**. Khi đó có thể đưa cùng một nhóm input cho Prompt, model hoặc knowledge base mới, rồi so sánh correctness, task completion rate, latency và cost.

Chỉ lưu một user question thì về sau rất khó tái hiện chính xác. Tối thiểu còn phải lưu version và snapshot của external dependency. Với dữ liệu động như order thực tế, giá cổ phiếu, có thể lưu tool result tại thời điểm đó hoặc data version có thể truy vết; không thể trông chờ vài ngày sau gọi lại interface sẽ nhận được cùng một câu trả lời.

Về Golden Set, Agent Evaluation, việc đưa production Badcase trở lại và release gate, có thể đọc tiếp [“Đánh giá ứng dụng LLM thế nào?”](../llm-basis/llm-evaluation.md).

## Kết nối Observability trong project Java/Spring AI thế nào?

Spring AI đã cung cấp Observability cho `ChatClient`, Advisor, `ChatModel`, `EmbeddingModel`, `ImageModel`, tool call và `VectorStore`. Khi kết nối project, trước tiên hãy thông suốt Trace cơ bản, sau đó bổ sung business Span; Evaluation và Audit có thể kết nối sau.

Config dưới đây lấy **Spring AI 2.0.0 và config property hiện tại của Spring Boot 4.x** làm ví dụ. Dependency và property có thể được điều chỉnh giữa các version khác nhau, khi upgrade nên lấy tài liệu chính thức của version tương ứng làm chuẩn.

### Làm thế nào để thông suốt Trace tích hợp sẵn của Spring AI?

Spring Boot 4 có thể sử dụng `spring-boot-starter-opentelemetry`, gửi Trace qua OTLP đến OpenTelemetry Collector hoặc backend tương thích. Ví dụ config quan trọng phía application như sau:

```yaml
management:
  tracing:
    sampling:
      probability: 0.1
    export:
      otlp:
        enabled: true
  opentelemetry:
    tracing:
      export:
        otlp:
          endpoint: http://otel-collector:4318/v1/traces

spring:
  ai:
    chat:
      client:
        observations:
          log-prompt: false
          log-completion: false
      observations:
        log-prompt: false
        log-completion: false
        include-error-logging: false
    tools:
      observations:
        include-content: false
    vectorstore:
      observations:
        log-query-response: false
```

Sampling probability mặc định hiện tại của Spring Boot là 0.1. Trong môi trường development, để kiểm tra cấu trúc Trace có thể tạm đổi thành 1.0; production cần xây sampling strategy dựa trên traffic, kích thước một Trace và capacity của backend.

Các content logging switch ở trên đều được giữ tắt. Trước tiên xác nhận metadata và call chain đã đủ để điều tra vấn đề, sau đó bật từng field theo data classification sẽ an toàn hơn nhiều so với ngay từ đầu gửi mọi Prompt và result lên nền tảng Observability.

### Những Span nào cần tự bổ sung?

Auto-instrumentation của framework chủ yếu bao phủ các AI basic component, còn business orchestration vẫn cần tự ghi. Ví dụ:

- Một Agent Run hoàn chỉnh;
- Rewrite question, rerank và cắt context;
- Workflow routing, tổng hợp song song và degradation;
- Structured output validation và security check;
- Phê duyệt thủ công, tạm dừng và khôi phục task;
- Business result, chẳng hạn yêu cầu hoàn tiền có thực sự tạo thành công hay không.

Project Spring Boot có thể dùng trực tiếp Micrometer Observation API để bọc các giai đoạn nghiệp vụ:

```java
@Component
public class RagObservationService {

    private final ObservationRegistry observationRegistry;
    private final DocumentRetriever documentRetriever;

    public RagObservationService(ObservationRegistry observationRegistry,
                                 DocumentRetriever documentRetriever) {
        this.observationRegistry = observationRegistry;
        this.documentRetriever = documentRetriever;
    }

    public List<DocumentHit> retrieve(RagQuery query) {
        Observation observation = Observation
                .createNotStarted("ai.rag.retrieve", observationRegistry)
                .lowCardinalityKeyValue(
                        "ai.rag.strategy",
                        query.strategy().name()
                )
                .highCardinalityKeyValue(
                        "ai.rag.knowledge_base_id",
                        query.knowledgeBaseId().toString()
                );

        return observation.observe(() -> {
            List<DocumentHit> hits = documentRetriever.retrieve(query);
            observation.lowCardinalityKeyValue(
                    "ai.rag.result",
                    hits.isEmpty() ? "empty" : "hit"
            );
            observation.highCardinalityKeyValue(
                    "ai.rag.result_count",
                    Integer.toString(hits.size())
            );
            return hits;
        });
    }
}
```

Đoạn code này chỉ minh họa sự khác nhau giữa ranh giới Span và cardinality field. `ai.rag.result` chỉ có hai giá trị `empty` và `hit`, có thể tham gia metric aggregation; số lượng kết quả recall chỉ đưa vào Trace. Project thực tế còn cần thống nhất error type và tag naming, không đặt raw query và document content làm Tag mặc định.

Nếu chỉ muốn thêm Span cho một method ổn định, cũng có thể sử dụng `@WithSpan` do [OpenTelemetry Java Agent cung cấp](https://opentelemetry.io/docs/zero-code/java/agent/annotations/):

```java
@WithSpan("ai.output.validate")
public ValidationResult validate(AgentResponse response) {
    return outputValidator.validate(response);
}
```

Khi business code đã sử dụng Micrometer Observation, không nên lại tự tạo thêm một lớp OpenTelemetry Span trùng lặp cho cùng một thao tác. Trước tiên hãy thống nhất entry point của instrumentation trong project, tránh xuất hiện hai node có tên khác nhau nhưng nội dung giống nhau trong Trace.

### Xác minh kết quả kết nối thế nào?

Tối thiểu cần chuẩn bị một nhóm scenario integration test:

1. RAG Q&A bình thường;
2. Retry sau khi model timeout;
3. Tool trả về business failure;
4. Thực thi song song bằng thread pool hoặc virtual thread;
5. Model response dạng streaming;
6. Gọi downstream qua HTTP hoặc message queue;
7. Agent khôi phục sau khi tạm dừng;
8. Prompt và tool result chứa thông tin nhạy cảm.

Khi xác minh, không chỉ kiểm tra “platform có Trace”, mà còn phải xác nhận:

- Quan hệ cha con và timeline song song có chính xác hay không;
- Mỗi retry có Attempt độc lập hay không;
- Error status, business result và cancellation status có chính xác hay không;
- Có thể truy vấn Token, model version và document ID hay không;
- Sau khi qua thread và cross-service, `traceId` có nhất quán hay không;
- Sensitive field có được masking hoặc hoàn toàn không được report hay không;
- Sau sampling có vẫn giữ được call chain quan trọng đầy đủ hay không.

Tài liệu hiện tại của Spring Boot còn đặc biệt nói rõ `@SpringBootTest` không tự động config Tracing component chịu trách nhiệm report data. Khi môi trường test không thấy export result, trước tiên xác nhận đã config rõ component report hay chưa, đừng trực tiếp kết luận đó là vấn đề propagation của context.

## Nên chọn nền tảng AI Observability thế nào?

Trước khi chọn, hãy xác định rõ project thực sự thiếu gì. Các khả năng thường gặp có thể chia thành bốn nhóm:

1. **Khả năng APM chung**: distributed call chain, liên kết log, metric alert và monitoring hạ tầng;
2. **Khả năng LLM Trace**: hiển thị Prompt, model, Token, cost, truy vấn và tool call;
3. **Khả năng Evaluation**: dataset, scorer, so sánh experiment, quản lý Badcase và release gate;
4. **Khả năng governance**: masking, tenant isolation, permission, retention period và audit.

Nếu company đã có hệ thống OpenTelemetry và APM hoàn thiện, có thể để application thống nhất output OTLP, sau đó kết nối UI Observability hoặc Evaluation chuyên dụng khi cần. Như vậy network, database và AI call có thể nằm trong cùng một distributed Trace.

Nếu project vẫn đang ở giai đoạn validation, cũng có thể bắt đầu bằng Observability tích hợp sẵn của framework và một backend hỗ trợ OpenTelemetry, chạy thông suốt các tầng Span cần thiết. Gắn quá sớm với nhiều field riêng của platform sẽ khiến việc migration về sau khá khó khăn.

Với các service hoặc middleware có sẵn trong nước, có thể tham khảo:

- [Alibaba Cloud ARMS](https://help.aliyun.com/zh/arms/application-monitoring/developer-reference/llm-trace-field-definition-description) hiển thị field của Completion, RAG, Agent và Tool, đồng thời nói rõ trong đó có extension được bổ sung dựa trên convention OpenTelemetry GenAI;
- [Tencent Cloud Log Service](https://cloud.tencent.com/document/product/614/133517) đặt metric, call tree của Trace và Session view trong cùng một trang Agent Observability;
- [Bài viết tiếng Trung về SkyWalking 10.4](https://skywalking.apache.org/zh/2026-04-05-virtual-genai-monitoring/) tập trung vào Java client collection, TTFT, Token và cost. Những tài liệu này cho thấy cách sản phẩm cụ thể được triển khai, nhưng Span type và extension field trong đó vẫn thuộc implementation riêng, không thể trực tiếp xem là convention chung của OpenTelemetry.

Khi chọn còn phải xác nhận các câu hỏi sau:

- Có hỗ trợ OpenTelemetry không, field nào thuộc private extension;
- Có thể liên kết HTTP, database và message queue Span thông thường hay không;
- Prompt và tool parameter có hỗ trợ masking, encryption và permission chi tiết hay không;
- Sampling rule có thể cấu hình theo error, latency, Token và business attribute hay không;
- Trace có thể thêm vào dataset và dùng cho Evaluation offline hay không;
- Data có thể được tenant isolation, xóa và thiết lập retention period hay không;
- Khi platform gặp sự cố, có ảnh hưởng đến main business request hay không.

Việc export telemetry nên bất đồng bộ, có timeout và giới hạn queue. Khi backend Observability không khả dụng, có thể mất một phần telemetry thông thường, nhưng không được ngược lại làm sập main chain của Agent. Khi audit data cần độ tin cậy cao hơn, nên sử dụng persistence chain độc lập.

## Những lỗi nào dễ mắc nhất khi triển khai AI Observability?

### Chỉ ghi lại một lần gọi model

Một Agent Run thường chứa nhiều lần gọi model, truy vấn và tool call. Chỉ ghi một dòng “gọi model thành công” ở tầng ngoài cùng thì không thể định vị vấn đề, cũng không nhìn thấy loop và retry.

### Xem HTTP thành công là task thành công

Tool trả về 200 nhưng nghiệp vụ có thể đã từ chối; model output bình thường nhưng câu trả lời có thể không đạt. Transport status, execution status, business status và quality result phải được ghi riêng.

### Mặc định report toàn bộ content

Prompt và tool result đầy đủ thực sự thuận tiện cho debug, đồng thời mang đến rò rỉ thông tin nhạy cảm, index phình to và storage cost. Production mặc định ghi metadata, content chỉ bật theo scenario, quyền hạn và thời hạn.

### Đưa user input vào metric label

User ID, Session ID, question text và order number sẽ tạo time series cardinality cao. Chúng nên đi vào Trace hoặc log được kiểm soát, không thể trực tiếp trở thành label chung của Metrics.

### Phụ thuộc vào auto-instrumentation nhưng không kiểm tra context

Thread pool, async stream, message queue và HTTP client tự tạo đều có thể làm đứt chain. Auto-instrumentation chỉ giảm công việc thủ công, cuối cùng vẫn phải kiểm tra topology call bằng integration test.

### Trace được sample thì audit cũng mất theo

Trace dùng để điều tra, có thể sampling. Audit record của thao tác nghiệp vụ rủi ro cao chịu trách nhiệm truy vết, không thể để random sampling của Trace thông thường quyết định có lưu hay không.

### Thu thập rất nhiều Trace nhưng không đưa Badcase trở lại

Nếu Badcase không đi vào dataset, sau khi sửa cũng không có Evaluation hồi quy, team chỉ có thể liên tục xử lý lại các vấn đề tương tự. Tối thiểu phải kết nối abnormal Trace, Evaluation set và release gate.

## Trả lời câu hỏi phỏng vấn liên quan đến AI Observability thế nào?

### Khác biệt lớn nhất giữa AI Observability và Observability truyền thống là gì?

Observability truyền thống chú trọng service, request và resource hơn, còn AI Observability phải bao phủ thêm model, Prompt, Token, truy vấn, tool và quá trình thực thi Agent. Vì output AI có tính xác suất, interface thành công cũng không thể chứng minh kết quả đúng, nên còn phải kết hợp Trace với Evaluation.

### Vì sao chỉ Metrics và log vẫn chưa đủ?

Metrics phù hợp để phát hiện bất thường tổng thể, log phù hợp để ghi event rời rạc. Một request Agent có thể chứa nhiều lần gọi model và tool, chỉ Trace mới có thể khôi phục tương đối trực quan thứ tự, quan hệ song song, latency và dependency upstream/downstream của chúng.

### Nên tách Span của Agent Trace thế nào?

Trước tiên tạo root Span cho một Run, sau đó tách child Span theo các giai đoạn quan trọng như gọi model, truy vấn, rerank, tool call, child Agent và validation kết quả. Chỉ tạo Span riêng cho vấn đề cần thống kê, retry, timeout hoặc định vị độc lập; không cần ghi lại toàn bộ method thông thường.

### Định vị câu trả lời sai của một RAG thế nào?

Kiểm tra rewrite question, document được recall, thứ tự rerank, cắt context, version Prompt và model output dọc theo Trace, sau đó dùng citation check, Evaluation faithfulness và các Evaluation khác để xác định câu trả lời có dựa trên tài liệu hay không. Cách này có thể phân biệt vấn đề truy vấn với vấn đề generation.

### Làm thế nào để giữ call chain khi nhiều Agent thực thi song song?

Để Span của các Agent con kế thừa cùng một workflow parent context. Khi cross-thread, hãy propagation Context tường minh hoặc sử dụng khả năng propagation context do framework cung cấp; khi cross-service và qua message queue, inject và extract W3C Trace Context. Giai đoạn aggregation còn phải ghi lại thực tế đã sử dụng kết quả của các Agent con nào.

### Có nên ghi toàn bộ Prompt và model answer vào Trace không?

Trên production không khuyến nghị mặc định lưu toàn bộ. Trước tiên ghi version, Token, latency, hash và result status; khi cần điều tra content thì tạm thời bật cho environment được kiểm soát hoặc request được chỉ định, đồng thời kết hợp masking, permission, encryption, retention period và audit.

### Chọn head sampling hay tail sampling thế nào?

Head sampling đơn giản, cost dễ kiểm soát, nhưng lúc request mới bắt đầu chưa biết có bất thường hay không. Tail sampling có thể ưu tiên giữ error, request chậm và Trace Token cao, đổi lại Collector phải chờ và cache data. Project thực tế thường dùng base sampling tỷ lệ thấp, sau đó tăng retention rate cho request bất thường và rủi ro cao.

### Quan hệ giữa Trace và Evaluation là gì?

Trace khôi phục “hệ thống đã làm gì”, còn Evaluation phán đoán “làm có tốt không”. Production failure Trace có thể được masking rồi tích lũy thành Badcase để replay offline, so sánh version và regression release.

## Tổng kết

Làm tốt AI Observability nghĩa là có thể khôi phục đầy đủ một request AI. Cài thêm một dashboard monitoring không thể giải quyết việc thiếu data và call chain bị đứt.

Một solution có thể thực sự dùng để điều tra production tối thiểu phải đạt các điểm sau:

- Phân biệt Session, Run, Trace, Span và Attempt;
- Bao phủ model, RAG, tool, Agent và business result cuối cùng;
- Propagation chính xác Trace context qua thread, service và message queue;
- Xử lý phân tầng field cardinality cao/thấp, metadata và content nhạy cảm;
- Kết hợp head sampling và tail sampling để kiểm soát cost, đồng thời giữ lại exception quan trọng;
- Đưa production Trace trở lại Badcase, Evaluation set và release gate;
- Xây dựng riêng Observability Trace và business Audit.

Khi production lại xuất hiện tình huống “interface thành công nhưng câu trả lời sai”, một log 200 còn xa mới đủ. Lần theo Trace để tìm model, tài liệu, tool, version và kết quả thực thi của từng bước tại thời điểm đó, hệ thống Agent mới có nền tảng để liên tục điều tra và cải tiến.

## Tài liệu tham khảo

- [OpenTelemetry: Generative AI Semantic Conventions](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/README.md)
- [OpenTelemetry: Semantic Conventions for GenAI Agent Spans](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-agent-spans.md)
- [OpenTelemetry: Semantic Conventions for GenAI Metrics](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-metrics.md)
- [OpenTelemetry: Java API và Context Propagation](https://opentelemetry.io/docs/languages/java/api/)
- [OpenTelemetry: Java Agent Instrumentation Annotations](https://opentelemetry.io/docs/zero-code/java/agent/annotations/)
- [OpenTelemetry Collector: Tail Sampling Processor](https://explorer.opentelemetry.io/collector/components/contrib-tailsamplingprocessor?type=processor)
- [W3C: Trace Context](https://www.w3.org/TR/trace-context/)
- [Spring AI: Observability](https://docs.spring.io/spring-ai/reference/observability/index.html)
- [Spring Boot: Tracing](https://docs.spring.io/spring-boot/reference/actuator/tracing.html)
- [OpenAI Agents SDK: Tracing (tiếng Trung giản thể)](https://openai.github.io/openai-agents-python/zh/tracing/)
- [Microsoft: Observability for Generative AI and agentic AI systems](https://learn.microsoft.com/en-us/security/zero-trust/sfi/observability-ai-systems)
- [Microsoft Foundry: Thiết lập tracing trong Microsoft Foundry](https://learn.microsoft.com/zh-cn/azure/foundry/observability/how-to/trace-agent-setup)
- [Alibaba Cloud ARMS: Mô tả định nghĩa field LLM Trace](https://help.aliyun.com/zh/arms/application-monitoring/developer-reference/llm-trace-field-definition-description)
- [Tencent Cloud Log Service: Chi tiết ứng dụng Agent Observability](https://cloud.tencent.com/document/product/614/133517)
- [Apache SkyWalking: Monitoring ứng dụng LLM dựa trên SkyWalking 10.4](https://skywalking.apache.org/zh/2026-04-05-virtual-genai-monitoring/)
