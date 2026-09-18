---
title: Model Context Protocol (MCP) là gì? Quan hệ với Function Calling và Agent?
description: Khái niệm cốt lõi của MCP (Model Context Protocol), kiến trúc phân tầng bốn lớp, cơ chế giao tiếp JSON-RPC 2.0 và thực tiễn phát triển MCP Server cấp production.
category: Phát triển ứng dụng AI
head:
  - - meta
    - name: keywords
      content: MCP,Model Context Protocol,JSON-RPC,Function Calling,AI Agent,tích hợp công cụ,Anthropic
---

Khi cùng một Git tool được kết nối với Claude Desktop, Cursor và Agent tự xây dựng, thường phải viết một lớp adapter cho mỗi nơi. Khi tham số tool, phương thức xác thực hoặc version thay đổi, nhiều client kết nối với nó cũng phải thay đổi theo.

MCP quy ước để hệ thống bên ngoài expose capability dưới dạng Server; Host hỗ trợ protocol này dùng Client để discover và gọi các capability đó. Nó xử lý việc tích hợp tool và data source; cách model quyết định gọi, cách task được orchestration vẫn thuộc trách nhiệm của Function Calling và Agent.

![Sơ đồ MCP](https://oss.javaguide.cn/github/javaguide/ai/skills/mcp-simple-diagram.png)

> Bài viết này lấy [2025-11-25 revision](https://modelcontextprotocol.io/specification/2025-11-25) hiện đang stable làm trọng tâm. Version 2025-03-26 điều chỉnh transport HTTP+SSE ban đầu thành Streamable HTTP, version 2025-06-18 bổ sung Elicitation, còn version 2025-11-25 bổ sung Tasks mang tính experimental và các nội dung như Elicitation dạng URL. Client và SDK có thể chỉ implement một phần trong số này; trước khi tích hợp cần đồng thời xác nhận protocol revision, version của SDK và capability của Host.

## MCP rốt cuộc là gì?

Tên đầy đủ của MCP là Model Context Protocol, tiếng Việt thường gọi là “model context protocol”.

Tách tên đầy đủ của MCP ra sẽ thấy khá rõ:

- Model: hướng đến ứng dụng LLM;
- Context: đưa context, tool và data source bên ngoài đến model;
- Protocol: quy định cách tương tác bằng một protocol tiêu chuẩn.

Tuy nhiên, cũng đừng hiểu MCP đơn giản là thêm plugin cho model. Trước đây khi xem mọi người thảo luận về MCP trong nhóm cộng đồng, khá nhiều bạn cũng nghĩ như vậy.

Nói chính xác hơn, MCP là **communication protocol giữa MCP Client và MCP Server**. Host chịu trách nhiệm chứa user interaction và model call, Client chịu trách nhiệm giao tiếp với Server, còn Server chịu trách nhiệm expose capability cụ thể.

Hãy lấy một scenario rất phổ biến.

Một người bạn hỏi: “Giúp tôi xem commit gần nhất của project này đã thay đổi gì.”

Model hoặc Agent bạn dùng dĩ nhiên không biết commit record của Git repository trên máy local. Nó phải nhờ capability bên ngoài để đọc Git log.

Không có MCP, mỗi AI application phải tự định nghĩa một cách riêng để “kết nối Git tool thế nào, truyền parameter ra sao, lấy result thế nào”.

Sau khi có MCP, capability liên quan đến Git có thể được đóng gói thành một MCP Server. MCP Client trong Host kết nối với nó, trước tiên discover các tool hiện có, sau đó gọi tool theo protocol, cuối cùng đưa result cho model tiếp tục phân tích.

Việc protocol adapter của Git tool được tập trung ở phía Server; Agent hoặc AI application chỉ cần hiểu user question, chọn tool và tổ chức result. Hai bên không cần định nghĩa lại một private interface cho từng client.

## MCP, Function Calling và Agent rốt cuộc có quan hệ gì?

Trong một task “đọc commit mới nhất của repository”, Function Calling, MCP và Agent có thể cùng xuất hiện nhưng lần lượt nằm ở các vị trí khác nhau: model trước tiên đưa ra call intent có cấu trúc, Host sau đó gửi nó đến tool thực tế, còn Agent quyết định có tiếp tục dựa trên result trả về hay không.

Lấy call intent do model output làm ví dụ:

```json
{
  "name": "read_file",
  "arguments": {
    "path": "/repo/README.md"
  }
}
```

OpenAI gọi cơ chế này là Function Calling, còn Anthropic gọi là Tool Use. Model nhờ đó output structured data như “gọi `read_file`, parameter là path này”.

MCP chịu trách nhiệm kết nối intent này với hệ thống bên ngoài: discover tool từ Server nào, truyền request ra sao và trả result về thế nào.

Agent quan tâm đến bước tiếp theo của task. Nó đọc tool result, tiếp tục gọi, kết thúc task hoặc chờ người dùng xác nhận; planning, memory và loop cũng thuộc layer này.
![Sơ đồ quan hệ ba lớp FC/MCP/Agent](https://oss.javaguide.cn/github/javaguide/ai/skills/mcp-fc-agent-layer.png)

Đặt ba thành phần vào một request chain sẽ trực quan hơn: Function Calling tạo command, MCP truyền command và kết nối tool, Agent quyết định khi nào chain tiếp tục và khi nào kết thúc.

Trọng tâm trong các scenario khác nhau như sau:

| Scenario                                       | Thành phần quan trọng hơn | Lý do                                                               |
| ---------------------------------------------- | ------------------------- | ------------------------------------------------------------------- |
| Để model quyết định có tra thời tiết hay không | Function Calling          | Trọng tâm là model chuyển intent thành parameter có cấu trúc        |
| Để Claude Desktop đọc file local               | MCP                       | Trọng tâm là có interface tiêu chuẩn giữa host và local file system |
| Để AI tự động điều tra sự cố production        | Agent                     | Trọng tâm là quyết định nhiều bước, tool call và phản hồi result    |

Trong project thực tế, ba thành phần thường xuất hiện cùng nhau; bảng chỉ dùng để phân biệt boundary trách nhiệm chính.

## MCP thực sự có những thành phần nào?

Communication chain của MCP gồm Host, Client và Server.

![Kiến trúc bốn lớp của MCP](https://oss.javaguide.cn/github/javaguide/ai/skills/mcp-four-layer-architecture.png)

Host là AI application mà người dùng sử dụng, ví dụ Claude Desktop, Cursor, AI plugin trong VS Code hoặc Agent platform tự xây dựng.

Client nằm bên trong Host, chịu trách nhiệm thiết lập session với MCP Server và trao đổi protocol message. Một Host có thể kết nối nhiều Server, thông thường mỗi Server tương ứng với một Client session.

Developer chủ yếu viết Server. Các capability như đọc file, query SQL, query GitHub Issue và query internal ticket đều có thể được nó expose cho Host.

Phía sau Server mới là data source thực tế: local file, database, internal platform, GitHub hoặc third-party API. Chúng không thuộc vai trò protocol của MCP. Host chỉ gọi Server thông qua Client; các implementation tầng dưới như query database và request API được xử lý bên trong Server.

## Một MCP call thường diễn ra thế nào?

Vẫn lấy “phân tích commit mới nhất của repository này” làm ví dụ.

![Sơ đồ sequence của MCP call](https://oss.javaguide.cn/github/javaguide/ai/skills/mcp-call-seq.png)

Sau khi nhận ra mình thiếu Git log, model trước tiên tạo tool call. Host giao call cho MCP Client, Client request đến Server thông qua JSON-RPC; Server query Git, sau đó trả result về theo path ban đầu, model dựa vào đó tổ chức câu trả lời.

Tên tool, `description`, mô tả parameter và scenario bị disable sẽ ảnh hưởng trực tiếp đến lựa chọn của model. Parameter Server nhận được cũng phải được xem là untrusted input: đọc file phải giới hạn directory, SQL phải parameterize, thao tác nguy hiểm phải approval, data trả về phải masking.

Còn một bước dễ bị bỏ qua: trước khi Client và Server chính thức gọi tool, chúng phải hoàn tất initialization handshake. Client gửi request `initialize`, kèm protocol version và danh sách capability mà nó hỗ trợ; Server trả về protocol version, capability và thông tin cơ bản mà nó hỗ trợ. Sau khi xác nhận, Client mới gửi notification `initialized`; hai bên khi đó mới chuyển sang trạng thái sẵn sàng.

Ý nghĩa của bước này là: Client biết được Server hỗ trợ capability nào (chỉ Tools? Hay còn Resources và Prompts?), Server cũng biết giới hạn của Client. Nhiều vấn đề kiểu “Server đã config nhưng tool không xuất hiện” khi troubleshoot đều nên kiểm tra trước xem initialization stage có thất bại hay không.

## MCP expose chỉ có Tools thôi sao?

Khi nói về MCP trong các nhóm kỹ thuật, nhiều reader chỉ nói về Tools. Điều này cũng bình thường vì tool call trực quan nhất. Nhưng MCP không chỉ có tool.

### Resources, Tools và Prompts

Server có thể cung cấp ba loại capability là Resources, Tools và Prompts.

Resources dùng để cung cấp context read-only, ví dụ local file, log fragment, database Schema hoặc config record.

Tools dùng để thực hiện action, ví dụ query database, gửi message, tạo ticket hoặc gọi business interface. Capability chủ động thực thi logic và có thể thay đổi external state nên được đặt trong Tools.

Prompts là prompt template có thể reuse, ví dụ “review code theo convention của team” hoặc “sắp xếp interface document thành test case”.

Tools thường do model lựa chọn và gọi; cách Resources và Prompts được hiển thị, lựa chọn vẫn có thể do Host, UI hoặc application logic quyết định.

Dùng một ví dụ đời thường để hiểu Resources, Tools và Prompts.

Một người bạn nói: “Tôi muốn ăn dưa chuột trộn.”

LLM đóng vai đầu bếp. Nó biết đại khái cách làm dưa chuột trộn, nhưng vẫn cần điều kiện bên ngoài:

- Resources giống nguyên liệu và công thức, chẳng hạn trong tủ lạnh có gì, ở nhà có dưa chuột không, gia vị để ở đâu;
- Tools giống các action cụ thể, chẳng hạn cắt rau, trộn gia vị, bật bếp, đặt mua nguyên liệu;
- Prompts giống preference cố định trong nhà, chẳng hạn ít cay, nhất định phải có rau mùi, không được cho tỏi.

Nếu tool description viết sai, ví dụ mô tả “dưa chuột” thành “cà chua”, model có thể chọn nhầm.

Đưa vào production environment, tên tool, parameter description và result structure đều ảnh hưởng trực tiếp đến lựa chọn và phán đoán tiếp theo của Agent. Server khởi động được mới chỉ là bắt đầu; boundary capability còn phải được model hiểu chính xác.

### Roots, Sampling và Elicitation

Ngoài capability phía Server, phía Client cũng có thể cung cấp một số capability để Server sử dụng, chẳng hạn Roots, Sampling và Elicitation.

Roots do Host thông qua Client cho Server biết: session hiện tại dự kiến làm việc trong những file system root directory nào. Ví dụ, Host chỉ có thể công bố project directory hiện tại mà không công bố home directory của user. Đây là capability negotiation và scope hint, không tự động tạo file system sandbox; Server vẫn phải thực hiện path normalization, boundary check và permission isolation ở cấp operating system.

Sampling khá đặc biệt, cho phép Server request LLM phía Host thực hiện một generation. Chẳng hạn sau khi Server đọc được một đoạn log, nó muốn nhờ model tóm tắt hoặc phân loại.

Elicitation là capability để Server hỏi bổ sung thông tin từ user trong quá trình thực thi. Ví dụ parameter chưa đầy đủ, option mơ hồ hoặc cần user confirmation trước khi execute thì có thể do phía Host hiển thị interaction.

Các capability này cần được chọn theo scenario. Phần lớn MCP Server có thể trước tiên chỉ cung cấp Tools; khi cần read-only context hoặc task entry có thể reuse mới cân nhắc Resources và Prompts. Roots, Sampling, Elicitation và Tasks còn phụ thuộc vào việc Client tương ứng có implement hay không; không thể chỉ nhìn vào việc Server SDK có interface hay không.

## Vì sao MCP dùng JSON-RPC?

Communication tầng dưới của MCP sử dụng JSON-RPC 2.0.

REST thiên về resource, ví dụ `/users/1`, `/orders/100`. JSON-RPC thiên về method call, ví dụ `tools/call`, `resources/read`. Tool call của AI vốn là “tôi muốn thực hiện một action nào đó”, nên JSON-RPC khá phù hợp với scenario sử dụng của MCP.

Một tool call request đại khái như sau:

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "read_file",
    "arguments": {
      "path": "/path/to/file.txt"
    }
  },
  "id": 1
}
```

Response có thể như sau:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Nội dung file..."
      }
    ]
  }
}
```

Chỉ khi thất bại mới trả về `error`:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32602,
    "message": "Invalid params"
  }
}
```

Response thành công chỉ trả về `result`, response thất bại mới trả về `error`; không thêm `error: null` vào response thành công.

Message của JSON-RPC ở dạng text, thuận tiện cho việc ghi log và không bị ràng buộc vào transport cụ thể. Cái giá phải trả là nó không có IDL mạnh và type constraint ở compile time như gRPC. Tuy MCP có thể dùng JSON Schema để mô tả tool parameter, Schema vừa là rule validation ở runtime vừa là gợi ý cho model; Server vẫn cần validate nghiêm ngặt mọi parameter.

## Chọn stdio hay Streamable HTTP thế nào?

Server local thường dùng stdio. Host khởi động nó dưới dạng sub-process, sau đó trao đổi message qua stdin/stdout; nhiều Server local trong Claude Desktop dùng cách này. Cách này không có chi phí deploy network bổ sung, nhưng Server chạy trên máy local nên phải siết riêng permission đối với file, Shell và database.

Nếu là Server bên thứ ba, tốt nhất đừng chạy trực tiếp khi chưa kiểm tra. Ít nhất hãy xem source code trước, hoặc dùng Docker, cgroups, namespace để isolation. Đặc biệt với Server liên quan đến file system, Shell và database, một khi cấp permission quá rộng thì rất khó bổ sung hạn chế về sau.

Ở mode stdio, stdout là channel của JSON-RPC message, không được dùng để in debug log. Chỉ một dòng `print()` cũng có thể phá hỏng message format, khiến Host parse thất bại hoặc Server disconnect. Debug log nên ghi vào stderr hoặc file; khi troubleshoot “Server khởi động thất bại”, cũng phải xác nhận stdout không bị trộn log.

Server remote phù hợp hơn với Streamable HTTP. Transport remote thời kỳ đầu của MCP thường dùng HTTP + SSE, sau đó dần chuyển sang Streamable HTTP. Sau khi message được thu về một endpoint thống nhất, authentication, load balancing và gateway integration có thể dùng lại cách vận hành của HTTP service thông thường.

```http
POST /mcp
Authorization: Bearer xxx
```

Response có thể là JSON thông thường hoặc SSE stream, tùy thuộc request type.

Khi chọn transport, có thể phán đoán theo vị trí deploy và phạm vi truy cập:

- Tool local, file local, sử dụng cá nhân: ưu tiên stdio.
- Team service, remote API, multi-user access: ưu tiên Streamable HTTP.
- Khi liên quan đến write operation và sensitive data, bất kể transport nào cũng phải bổ sung authentication, rate limiting và audit.

![Chọn transport của MCP](https://oss.javaguide.cn/github/javaguide/ai/skills/mcp-transport-decision.png)

## Ý nghĩa của MCP chỉ là khiến model biết gọi interface sao?

Function Calling đã có thể giúp model biểu đạt “gọi interface nào”. MCP giải quyết việc một tool giống nhau được deliver đến nhiều Host mà không phải tích hợp lặp lại.

Ví dụ, sau khi internal ticket system được tích hợp vào một Agent, khi đổi sang Host khác thường vẫn phải viết lại phần connection, parameter và result handling. Sau khi đóng gói capability thành MCP Server, các application hỗ trợ MCP Client có thể tích hợp theo cùng một cách discover và call.

Boundary này tương tự việc frontend và backend phối hợp thông qua interface contract: Agent development tập trung vào task và interaction, tool development tập trung vào capability implementation, data permission và operation boundary.

Trong team, operation manual, on-call document, incident review và troubleshooting script thường phân tán trong document library, Wiki hoặc script repository. Sau khi sắp xếp các capability query và troubleshoot có thể cấp quyền thành Server, Agent mới có thể đọc document, đọc config hoặc chạy tool trong phạm vi đã định, thay vì chỉ đưa ra một đoạn giải thích chung chung.

## Sau khi tích hợp MCP thì có thể đưa thẳng lên production không?

Không. Chain “cài một Server, hỏi một câu, nhận result” trong Demo rất ngắn; production environment cần bổ sung đầy đủ interface constraint, audit và operation governance.

Time field là ISO-8601 hay timestamp, đơn vị tiền là đồng hay phân, default value của pagination là gì, tất cả đều phải được ghi vào Schema, field description và example. Server phải dựa vào đó để validate parameter và trả về error message đủ để model sửa request.

Một câu trả lời của Agent có thể đi qua nhiều Server và tool. Trace ID, structured log và call chain cần ghi lại call parameter, duration, result summary và error code thì mới xác định được bước nào ảnh hưởng đến câu trả lời cuối cùng.

stdio local có thể nhận được file permission trên máy user, còn Server remote có thể kết nối tới internal system. File directory, table có thể query, việc có được write production API hay không, việc có được gửi email hay không đều phải được cấp quyền rõ ràng. Write operation như delete, modify, send và production call còn cần second confirmation, audit và rollback plan.

`description` của Server, Prompt template và content trả về cũng cần được review: content độc hại hoặc sơ sài có thể chèn prompt injection, dẫn dắt model đọc thêm file hoặc gửi thông tin ra ngoài. Nguồn Server, dependency package, permission scope và update record đều thuộc phạm vi review trước khi đưa lên production.

Token của model, vector retrieval, third-party API và cloud resource đều phát sinh chi phí. Call phải có thể liên kết với user, business line và tool; nếu không, khi chi phí tăng sẽ không biết cost đến từ đâu.

Field, enum hoặc result structure của tool interface nếu có breaking change cũng sẽ thay đổi phán đoán của model. Tool-level version, gradual rollout, giữ version cũ và automated compatibility test nên được maintain cùng Server.

## Trước khi đưa MCP vào doanh nghiệp, nên kiểm tra những vấn đề nào?

### Schema và version

- Mỗi tool đã có input/output Schema rõ ràng chưa?
- Đơn vị field, time format, enum value và default value đã được viết rõ chưa?
- Tool interface đã có version number chưa?
- Với breaking change đã có gradual rollout và rollback plan chưa?
- Có thể thực hiện automated validation dựa trên Schema không?

### Permission và security

- Server có thể truy cập những file, directory, database và API nào?
- Có phân biệt read-only tool và write operation không?
- High-risk operation có cần human confirmation không?
- Result trả về đã được masking chưa?
- Có phòng chống path traversal, SQL injection và command injection không?
- Third-party MCP Server đã qua review source code, dependency và permission chưa?

### Observability

- Mỗi user request đã có Trace ID chưa?
- Tool call parameter, duration, result summary và error code đã có structured log chưa?
- Có thể khôi phục đầy đủ tool call chain phía sau một câu trả lời của Agent không?
- Đã có strategy timeout, rate limiting, circuit breaker và retry chưa?

### Cost attribution

- Mỗi call có thể liên kết với user, business line, tool và session không?
- Chi phí Token, API và cloud resource có thể được thống kê tách riêng không?
- Đã có quota và budget alert chưa?
- Khi model loop gọi tool, đã có giới hạn số lần call chưa?

### Dependency governance

- MCP SDK, third-party library và third-party Server có maintainer và update record không?
- Ai chịu trách nhiệm theo dõi security vulnerability?
- Server upgrade đã có test environment và rollback strategy chưa?
- Có tránh đặt capability cốt lõi vào extension bên thứ ba không có maintainer không?

Các hạng mục kiểm tra này về bản chất không khác ordinary backend service. MCP thay đổi cách tích hợp tool, không thay thế authentication, audit, log, version và rate limiting.

## Khi viết MCP Server cần lưu ý điều gì?

### Đừng theo đuổi lớn và đầy đủ ngay từ đầu

Một lỗi thường gặp của Server là dùng một số “universal tool” để chứa mọi operation:

```text
execute_sql(sql)
file_operation(op, path, data)
call_api(url, method, body)
```

Các interface như `execute_sql`, `file_operation` giao cả operation scope và permission cho model đoán; càng nhiều parameter thì không gian misuse và privilege escalation càng lớn. Sau khi tách theo business action, Schema, permission và audit rule mới có thể lần lượt gắn vào từng tool cụ thể:

```text
get_user_by_id(id)
list_active_orders(user_id)
read_file(path)
write_report(path, content)
```

Tên tool có thể dùng động từ cộng danh từ, còn `description` giải thích condition áp dụng, required parameter và scenario bị disable.

Ví dụ, tool query slow SQL ngoài việc “query slow SQL log” còn nên ghi rõ: dùng khi service response chậm, database timeout, CPU tăng vọt và nghi ngờ liên quan đến database; không gọi khi user hỏi vấn đề network hoặc memory. Constraint như vậy có thể giảm trường hợp model gửi nhầm vấn đề tương tự đến tool sai.

### Cẩn thận với file lớn và text dài

Log, Markdown document, web HTML và CSV file có thể vượt xa context của model. Resource interface có thể trước tiên trả về file name, size, update time, summary và readable range; khi cần content thì đọc theo chunk.

Có thể giới hạn một chunk ở khoảng 100KB; khi resource vượt 10MB thì chỉ trả về mô tả và cách đọc tùy chọn, không trả trực tiếp toàn bộ content. Cách này vừa tránh lấp đầy context trong một request, vừa ngăn Server tiêu tốn quá nhiều memory hoặc network resource do file lớn.

Đừng gắn limit với tokenizer của một model cụ thể. Cách tính token của các model khác nhau; Server chỉ cần dùng character count hoặc byte count để kiểm soát ở mức thô, còn việc cắt context do Host hoặc application tầng trên phụ trách.

### Không thể giải quyết security issue bằng cách tin model

Sau khi normalize path, việc đọc file phải kiểm tra directory boundary, không được để `../` vượt khỏi phạm vi cho phép. SQL query phải dùng parameterized statement, không được execute trực tiếp string do model generate.

Data trả về như số điện thoại, email, Token, key và internal link cần được masking. Write operation như delete file, modify database, gửi email và gọi production interface mặc định phải siết permission, đồng thời thiết lập human confirmation và audit.

Khi model rơi vào loop, nó có thể lặp lại cùng một tool. Rate limiting, timeout, circuit breaker và quota phải được chính Server triển khai, không thể giả định Host chắc chắn sẽ xử lý thay.

Toàn bộ chain liên quan đến Prompt Injection, Token Passthrough, resource-level authentication, isolation của local Server và supply chain của third-party Skill/MCP có thể xem tiếp tại [Thực chiến security cho LLM/Agent](../system-design/llm-security.md).

### Ví dụ tối thiểu về MCP Server: chạy được một tool trước

Dùng official Python SDK viết một weather Server, đại khái như sau:

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("weather-server")

@mcp.tool()
def get_weather(city: str) -> str:
    """Lấy thông tin thời tiết của city được chỉ định"""
    return f"Hôm nay {city} trời nắng, nhiệt độ 25°C"

@mcp.resource("weather://forecast")
def weather_forecast() -> str:
    """Trả về dự báo thời tiết trong một tuần tới"""
    return "Dự báo thời tiết bảy ngày tới..."

if __name__ == "__main__":
    mcp.run()
```

Trong Claude Desktop có thể config như sau:

```json
{
  "mcpServers": {
    "weather-server": {
      "command": "uv",
      "args": ["run", "--with", "mcp", "/path/to/weather_server.py"]
    }
  }
}
```

Khi debug local, nên dùng trực tiếp MCP Inspector:

```bash
# Python Server
npx @modelcontextprotocol/inspector uv run --with mcp /path/to/weather_server.py

# Node Server
npx @modelcontextprotocol/inspector node build/index.js
```

Nó có thể mô phỏng Host gửi request. Server initialization có vấn đề không, tool có discover được không, parameter validation có báo lỗi không, về cơ bản đều có thể thấy trước ở đây.

Trong production environment, đừng phụ thuộc vào việc global `python` vừa hay đã cài `mcp`. Hãy dùng interpreter của virtual environment, hoặc explicit declaration dependency bằng `uv run --with mcp ...` như trên để ổn định hơn. Nếu Claude Desktop khởi động thất bại, trước tiên hãy xem `mcp.log`, đừng ngay lập tức nghi ngờ protocol có vấn đề; nhiều khi chỉ là path hoặc dependency chưa khớp.

## Dùng Inspector để verify Server remote

Ví dụ trên khởi động process local thông qua stdio. Để quan sát initialization, tool discovery và call của Server remote, có thể tiếp tục dùng [MCP Inspector](https://github.com/modelcontextprotocol/inspector), chuyển sang kết nối Streamable HTTP.

Ở đây lấy [Parallel Search MCP](https://docs.parallel.ai/integrations/mcp/search-mcp) làm ví dụ. Nó cung cấp web search `web_search` và web content extraction `web_fetch`; endpoint anonymous là `https://search.parallel.ai/mcp`, không cần Parallel account hoặc API Key, nhưng anonymous access bị rate limit.

Phần dưới dùng Inspector 2.5.0, cần Node.js 22.19.0 hoặc cao hơn. Trước tiên liệt kê tool trong terminal:

```bash
npx --yes @modelcontextprotocol/inspector@2.5.0 --cli \
  https://search.parallel.ai/mcp --transport http --method tools/list
```

`--transport http` chỉ định Streamable HTTP. Inspector trước tiên hoàn tất initialization rồi gửi `tools/list`; trong result trả về cần thấy `web_search`, `web_fetch` và Schema parameter của chúng. Ví dụ anonymous này không truyền request header `Authorization`.

Tiếp theo gọi một lần search tool để tìm tài liệu chính thức về Java virtual thread:

```bash
npx --yes @modelcontextprotocol/inspector@2.5.0 --cli \
  https://search.parallel.ai/mcp --transport http \
  --method tools/call --tool-name web_search \
  --tool-arg 'objective=Tìm tài liệu chính thức của OpenJDK về Java virtual thread' \
  --tool-arg 'search_queries=["OpenJDK JEP 444 virtual threads"]'
```

`objective` và `search_queries` đều là required parameter, trong đó parameter sau là JSON array. Tool result sẽ bao gồm source URL và content excerpt, có thể lần theo link để đối chiếu câu trả lời. Khi troubleshoot, cần phân biệt connection failure, JSON-RPC error và `isError: true` do tool trả về; chỉ nhận HTTP 200 không có nghĩa tool execute thành công.

Khi search hoặc extract, context như search term, target URL và target description được truyền vào sẽ gửi đến Parallel; chỉ sử dụng content phù hợp để giao cho third party xử lý. Sau khi CLI command này chạy xong, connection sẽ disconnect và không cài service cho Host khác; nếu sau đó enable các tool này trong Agent, Agent cũng có thể tự gọi theo nhu cầu của task, dùng xong có thể disable hoặc remove connection trong Host tương ứng.

## Ghi lại protocol revision khi tích hợp

MCP thống nhất cách Host discover và call external tool, data source, nhưng không thay thế business authentication, data permission và execution audit. Một Server dùng được trong một Host không có nghĩa khi chuyển sang Host khác vẫn hỗ trợ các capability tùy chọn như Sampling, Elicitation và Tasks.

Khi implement Server tối thiểu, trước tiên cố định protocol revision và SDK version, dùng Inspector để verify initialization, capability negotiation, parameter validation và error response. Sau khi chuẩn bị tích hợp remote service, tiếp tục bổ sung OAuth, rate limiting, Trace, version compatibility và rollback; với file và command tool, cũng phải triển khai directory validation và sandbox ở phía Server.

## Tổng kết

MCP thiết lập cách discover và call capability thống nhất cho Host, Client và Server, giải quyết vấn đề tích hợp external tool và data source. Cách model quyết định call, cách business orchestration và việc request có được authorize hay không lần lượt do Function Calling, Agent/Workflow và business security policy phụ trách.

Khi tích hợp, trước tiên xác nhận protocol revision, kết quả capability negotiation của SDK và Host, dùng Server tối thiểu để verify connection, Schema và error handling. Khi vào production environment, tool permission, data masking, directory hoặc SQL validation, rate limiting, audit và version compatibility đều cần Server và Host cùng triển khai.
