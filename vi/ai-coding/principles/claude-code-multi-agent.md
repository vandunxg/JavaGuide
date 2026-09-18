---
title: "Giải thích chi tiết cơ chế Claude Code Multi-Agent: Subagent, Subtask, Fork và Agent Teams"
description: "Kết hợp tài liệu chính thức của Claude Code và phân tích source code cộng đồng để hệ thống hóa Subagent, Subtask, Fork Session, Agent Teams, cộng tác nhiệm vụ, luồng ngược quyền hạn và kiểm soát chi phí, giúp hiểu cách cơ chế Claude Code Multi-Agent phân tách nhiệm vụ, cô lập context và quản lý cộng tác."
category: AI Coding Principles
tag:
  - Claude Code
  - Multi-Agent
  - AI Agent
  - AI Coding
head:
  - - meta
    - name: keywords
      content: Claude Code,Multi-Agent,Subagent,Subtask,Fork Session,Agent Teams,AI Agent,cô lập context,cộng tác nhiệm vụ,AI Coding
---

Xin chào, mình là Tiểu G. Gần đây có một người bạn hỏi mình: Subagent, Fork và Agent Teams trong Claude Code rốt cuộc có phải là một không? Nếu được hỏi về cơ chế Claude Code Multi-Agent trong phỏng vấn, nên trả lời thế nào?

Ban đầu mình cũng nghĩ đây chỉ là vài cái tên dễ gây rối. Nhưng khi thực sự đặt tài liệu chính thức, changelog và source code cộng đồng cạnh nhau để phân tích, mình mới nhận ra khác biệt không hề nhỏ.

Claude Code đơn Agent đã có thể xử lý khá nhiều việc: sửa code thường ngày, tìm vấn đề, bổ sung test, phần lớn thời gian là đủ.

Vấn đề là nhiệm vụ trong dự án thực tế thường không gọn gàng như vậy. Một Session vừa phải tìm kiếm, đọc, thử sai, vừa phải tạo ra thay đổi cuối cùng; nói chuyện một lúc thì context đã trở nên lộn xộn.

Ví dụ khi điều tra vấn đề performance của một API, nó có thể tìm API trước, rồi đọc Mapper, xem log, kiểm tra index, giữa chừng còn thử vài câu SQL. Đến lúc thực sự cần sửa code, lịch sử trò chuyện đã đầy những file không liên quan, phỏng đoán cũ và các phương án đã bị bác bỏ.

Người đọc còn thấy mệt, huống hồ model cũng dễ bị những thông tin quá trình này dẫn lệch hướng.

Điều Claude Code Multi-Agent tập trung vào chính là **vấn đề context và phân tách nhiệm vụ**.

Nó không dồn mọi việc cho một Session, mà tách theo tính chất nhiệm vụ:

- Giao việc tìm kiếm một lần cho Subagent;
- Giao việc khám phá nhánh đã có context cho `/subtask`, khi cần tiếp tục độc lập thì dùng `/fork`;
- Với nhiệm vụ cần nhiều người cộng tác, mới dùng Agent Teams.

Vì vậy mình đã tổng hợp các phần liên quan đến Multi-Agent trong Claude Code. Bài viết tham khảo phân tích source code Claude Code do cộng đồng tổng hợp để đi sâu vào nguyên lý, nhưng cách sử dụng hiện tại lấy tài liệu chính thức và changelog làm chuẩn.

Các tính năng này thay đổi rất nhanh. Thông tin phiên bản trong bài được đối chiếu đến Claude Code v2.1.218 (2026-07-24): từ v2.1.212, trong điều kiện thông thường, forked subagent trong Session hiện tại dùng `/subtask`, còn `/fork` sao chép cuộc trò chuyện hiện tại và tạo một Session nền độc lập. Khi tắt Agent View thì đây là ngoại lệ: `/subtask` không khả dụng, `/fork` tiếp tục khởi chạy forked subagent. Các bài viết cũ gọi cả hai hành vi này là Fork nên dễ gây nhầm lẫn.

## Vì sao Claude Code cần nhiều Agent?

Viết một function nhỏ, sửa một config, bổ sung một đoạn test thì một Agent thường là đủ.

Tuy nhiên khi thực hiện nhiệm vụ xuyên module, có thể sẽ không đủ. Ví dụ bạn yêu cầu Claude Code điều tra một truy vấn chậm trên production, nó có thể phải liên tục làm những việc sau:

- Tìm API và SQL liên quan;
- Đọc code tầng ORM / Mapper;
- Xem index và execution plan;
- Sửa logic truy vấn;
- Bổ sung test hoặc script benchmark;
- Cuối cùng tổng kết nguyên nhân và thay đổi.

Nếu để một Agent làm toàn bộ các bước này, dồn hết vào main Session, rắc rối chủ yếu nằm ở hai điểm:

- Quá nhiều thông tin quá trình. Các file không liên quan trúng kết quả tìm kiếm, log cũ, phương án thất bại và phỏng đoán tạm thời đều ở lại trong context. Khi tiếp tục viết code, model còn phải lọc trọng tâm hiện tại từ những tài liệu đã lỗi thời này.
- Quán tính nhiệm vụ. Vừa điều tra xong vấn đề database, lượt sau lại yêu cầu nó review frontend component, nó có thể vẫn mang theo cách phán đoán của lượt trước để xem vấn đề.

Vì vậy, cách mình hiểu về Multi-Agent là **trước hết phải bảo vệ main Session. Main Session chịu trách nhiệm phán đoán và triển khai, còn việc nặng, việc vụn, việc ở nhánh thì có thể tách là tách. Còn tăng tốc song song chỉ là sản phẩm phụ sau khi đã phân tách hợp lý.**

Ý tưởng này cũng giống với điều thường được nói trong context engineering là “cô lập quá trình ở nhánh”: main Session giữ lại phán đoán, kế hoạch và quyết định cuối cùng, còn việc tìm kiếm, xác minh, review dễ phình to thì giao cho worker độc lập.

![Subagent phân tách nhiệm vụ, cô lập context](https://oss.javaguide.cn/github/javaguide/ai/context-engineering/sub-agent-task-splitting-context-isolation%20.png)

Trước tiên hãy xem bảng mình tự tổng hợp. Khi nhìn vào Multi-Agent trong Claude Code, có thể phân biệt trước theo vài loại vấn đề:

| Vấn đề                                                | Cơ chế phù hợp | Giải thích                                                                                  |
| ----------------------------------------------------- | -------------- | ------------------------------------------------------------------------------------------- |
| Quá nhiều tìm kiếm ở nhánh, làm bẩn main Session      | Subagent       | Subagent tự đọc file, tra cứu tài liệu, main Session chỉ nhận kết quả                       |
| Cần kế thừa context hiện tại để khám phá nhánh        | `/subtask`     | Forked subagent trong Session hiện tại kế thừa context và trả kết quả                       |
| Cần sao chép cuộc trò chuyện và tiếp tục độc lập      | `/fork`        | Tạo một Session nền có thể khôi phục và quản lý độc lập                                     |
| Nhiều vai trò cần cộng tác, liên lạc và nhận nhiệm vụ | Agent Teams    | Mỗi teammate là một instance Claude Code độc lập, có task list và cơ chế message dùng chung |

> **Khác biệt môi trường**: Bảng này được tổng hợp theo trường hợp mặc định khi Agent View đã bật. Khi tắt Agent View, `/subtask` không khả dụng, `/fork` sẽ khởi chạy forked subagent trong Session hiện tại.

Tên đều có chữ Agent nhưng công việc khác nhau khá xa:

- Cách dùng Subagent gần với “bạn đi kiểm tra thử, xong thì báo lại”.
- `/subtask` sao chép context trong Session hiện tại để làm nhiệm vụ ở nhánh; `/fork` tạo một Session nền độc lập.
- Agent Teams cho phép vài instance độc lập cùng làm một dự án, có thể gửi message, nhận nhiệm vụ, rồi tổng hợp sau cùng.

![So sánh Subagents và Agent Teams](https://oss.javaguide.cn/github/javaguide/ai/coding/claude-code-subagents-vs-agent-teams.png)

Còn một chi tiết: không nhất thiết phải tự chỉ định Subagent. Tài liệu chính thức nói rằng Claude sẽ dựa vào `description` của Subagent để phán đoán khi nào ủy thác; các Subagent tích hợp sẵn như Explore, Plan, general-purpose cũng sẽ được tự động dùng cho nhiệm vụ phù hợp.

Sau khi xem bảng này, hãy hỏi một vấn đề thực tế hơn: rốt cuộc có nên chỉ định Subagent rõ ràng không, và khi nào nên nâng cấp lên Agent Teams.

Khi lựa chọn, trước tiên hãy hỏi: **các worker này có cần trao đổi với nhau không?**

Nếu chỉ cần kiểm tra xong rồi báo lại, Subagent là đủ. Code review, phân tích log, nghiên cứu một điểm đơn lẻ đều thuộc loại này.

Nếu vài worker cần gửi message cho nhau, nhận nhiệm vụ và trao đổi kết quả trung gian, mới cân nhắc Agent Teams. Ví dụ một teammate xem API backend, một teammate xem trang frontend, một teammate chuyên làm test và nghiệm thu.

Chi phí cũng rất thực tế: mỗi teammate có context window riêng, lượng token sẽ tăng theo số teammate đang hoạt động. Các nhiệm vụ nghiên cứu, review, phân tách tính năng mới thường đáng dùng; thay đổi nhỏ thường ngày thì một Session lại tiết kiệm hơn.

## Subagent: ủy thác nhẹ trong main Session

### Subagent là gì?

Subagent là một trong những cơ chế ủy thác được dùng phổ biến nhất và khó dùng quá mức nhất trong Claude Code.

Bạn có thể hiểu nó là một worker được main Session tạm thời phái đi. Nó có context window riêng và có thể dùng các tool được chỉ định. Khi nhiệm vụ kết thúc, nó trả kết quả về main Session, không đổ toàn bộ quá trình tìm kiếm trở lại.

Nhiều khi Agent đọc thêm vài file không phải vấn đề. Rắc rối là nó mang cả quá trình tìm kiếm, phán đoán tạm thời và phỏng đoán cuối cùng đã bị bác bỏ về main Session. Điểm tốt của Subagent nằm ở đây: để nó tự kiểm tra, main Session chỉ nhận kết quả đã được tổng hợp.

Mình thấy điểm này rất đáng giá: main Session không phải cùng bị kéo ra ngoài.

![Claude Code Explore Subagent: tìm kiếm ở nhánh chạy trong background](https://oss.javaguide.cn/github/javaguide/ai/coding/claude-code-explore-subagent-demo.png)

Trong hình trên, main Session chỉ giao việc tìm kiếm liên quan đến login, authentication và kiểm tra permission cho Explore subagent. Quá trình tìm kiếm chạy trong background, tuyến chính tiếp tục sạch, sau khi subagent kết thúc thì nhận danh sách file, call chain và các điểm cần theo dõi tiếp theo đã được tổng hợp.

![Claude Code Sub-Agent: giữ cuộc trò chuyện chính sạch](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode-sub-agent.png)

Khi nào cần custom Subagent?

Theo mình: khi cùng một loại worker lặp lại thường xuyên và mỗi lần đều phải đưa cho nó cùng một bộ instruction thì mới nên tích lũy thành custom Subagent. Ví dụ bạn thường xuyên yêu cầu nó review code chỉ đọc, kiểm tra truy vấn database, security scan, quy nguyên nhân từ log; vai trò, quyền tool và format output của các nhiệm vụ này khá ổn định, nên tách riêng một cái là đáng.

Nếu chỉ thỉnh thoảng kiểm tra một file hoặc xem tạm một đoạn log, cứ để Claude Code dùng Subagent tích hợp sẵn hoặc ủy thác thủ công là đủ, không cần vì “trông có vẻ chuyên nghiệp” mà tạo file riêng.

File custom thường nằm ở:

```text
~/.claude/agents/
.claude/agents/
```

Hai directory này không nhất thiết tồn tại mặc định. Không thấy chúng là bình thường, nghĩa là máy hoặc project hiện tại chưa từng tạo custom Subagent.

Cách tạo trong tài liệu phiên bản mới trực tiếp hơn: yêu cầu Claude viết giúp, hoặc tự tạo directory và viết file Markdown. Từ v2.1.198, `/agents` không còn mở wizard tạo tương tác, chỉ nhắc bạn yêu cầu Claude tạo hoặc trực tiếp chỉnh sửa `.claude/agents/`. Nếu đây là lần đầu tạo directory `agents` trong Session hiện tại, Claude Code có thể cần restart mới phát hiện được.

Subagent cấp user có hiệu lực với mọi project, Subagent cấp project phù hợp để chia sẻ với team.

File Subagent chính là Markdown + YAML frontmatter, có thể cấu hình name, description, tool, model, permission mode, hooks và skills. `name` và `description` là bắt buộc, trong đó `description` rất quan trọng vì Claude dựa vào nó để phán đoán khi nào tự động ủy thác.

### Subagent chạy như thế nào?

Từ running log hoặc phân tích source code cộng đồng, có vài parameter trong mỗi lần gọi `Agent` đáng chú ý nhất:

| Parameter           | Tác dụng                                                                                                                        |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `description`       | Tóm tắt nhiệm vụ dành cho main Session                                                                                          |
| `prompt`            | Nhiệm vụ cụ thể giao cho subagent thực hiện                                                                                     |
| `subagent_type`     | Chỉ định loại Subagent sử dụng; nếu bỏ qua vẫn là `general-purpose`, không tự động thành fork                                   |
| `model`             | Chỉ định alias model mà subagent sử dụng                                                                                        |
| `run_in_background` | Có chạy trong background hay không; phiên bản mới tự lựa chọn khi không cấu hình rõ, từ v2.1.198 mặc định chạy trong background |
| `name`              | Đặt tên có thể định vị cho Subagent chạy nền hoặc teammate của Agent Teams                                                      |
| `team_name`         | Field của Agent Teams phiên bản cũ; phiên bản mới vẫn nhận nhưng sẽ bỏ qua                                                      |

Bảng này thể hiện parameter gọi của tool `Agent`, không phải YAML frontmatter của `.claude/agents/*.md`. Các config tương ứng cho background và workspace trong file Subagent là những field như `background`, `isolation`.

Không cần học thuộc bảng này. Điểm chính là đừng dựa vào cách nói cũ rằng “bỏ qua `subagent_type` sẽ implicit fork”. Khi chưa chỉ định type, lời gọi Agent thông thường dùng `general-purpose`; nếu cần kế thừa context, phải chỉ định rõ `/subtask` hoặc config fork tương ứng do phiên bản hiện tại cung cấp.

Trong implementation nội bộ, `AgentTool` chịu trách nhiệm entry point và routing, còn `runAgent()` mới thực sự khởi chạy subagent.

`runAgent()` trước tiên sẽ thực hiện một loạt chuẩn bị runtime:

- Khởi tạo MCP Server mà agent cần;
- Tạo `ToolUseContext` dành riêng cho subagent;
- Chạy các hook liên quan đến `SubagentStart`;
- Ghi sidechain transcript và agent metadata;
- Đi vào vòng lặp chính `query()`.

Các chi tiết này cho thấy một điều: Subagent không phải một function call thông thường trong main Session. Nó tái sử dụng Agent runtime của Claude Code, có tool, permission, context, message flow và transcript.

Vì vậy, nó phù hợp để đảm nhận nhiệm vụ ở nhánh tương đối đầy đủ. Để nó đọc một nhóm file, thực hiện một lượt review và đưa ra kết luận sẽ sạch hơn nhiều so với dồn quá trình này vào main Session.

### Nhiệm vụ nào phù hợp để giao cho Subagent?

Mình thường giao các nhiệm vụ sau cho Subagent:

- Review chỉ đọc một module, cuối cùng đưa ra danh sách vấn đề;
- Tìm một loại error log, main Session chỉ nhận kết luận;
- Tổng hợp cách sử dụng một thư viện bên ngoài mà không mang quá trình tìm kiếm về;
- Xác minh độc lập một thay đổi, nếu thất bại cũng có thể ủy thác lại một lần nữa.

Những nhiệm vụ cần main Session liên tục tham gia phán đoán thì tốt nhất đừng tách ra.

Ví dụ đang sửa một file cốt lõi, main Session và subagent cùng bắt tay làm, cuối cùng rất có thể không tăng tốc mà còn tạo conflict. Thói quen của mình là để Subagent làm nhiều việc chỉ đọc và xác minh, hạn chế để nó trực tiếp tham gia thay đổi tuyến chính.

## `/subtask`, `/fork` và Agent chạy nền: khi nào kế thừa context?

### Khác biệt giữa `/subtask` và Subagent thông thường

Subagent thông thường thường bắt đầu làm việc sau khi main Session đưa một prompt rõ ràng. Mặc định không nên kế thừa toàn bộ lịch sử main Session, nếu không ý nghĩa của việc “cô lập thông tin quá trình” sẽ mất.

`/subtask` đi theo một hướng khác: nó khởi chạy forked subagent trong Session hiện tại, kế thừa context cuộc trò chuyện đã hình thành ở parent Session, rồi trả kết quả về tuyến chính hiện tại. Phải chủ động chọn hướng này; bỏ qua `subagent_type` không kích hoạt implicit fork.

Điều này phù hợp với một thời điểm khá đặc biệt: main Session vừa có một context chất lượng cao, bạn không muốn lãng phí nó nhưng lại muốn thử vài hướng.

Ví dụ main Session đã đọc xong toàn bộ module payment, lúc này bạn muốn tiện thể chia ra vài hướng để kiểm tra:

- Kiểm tra vấn đề trong thiết kế state machine;
- Kiểm tra vấn đề trong logic idempotent;
- Kiểm tra thiếu sót trong test coverage.

Khi đó `/subtask` phù hợp hơn Subagent thông thường. Mỗi child đều nhận được context mà parent Session vừa xây dựng, không cần đọc lại project từ đầu.

![Claude Code Fork: khởi chạy nhánh nền dựa trên context hiện tại](https://oss.javaguide.cn/github/javaguide/ai/coding/claude-code-fork-subagent-demo.png)

Hình trên ghi lại hành vi `/fork` của phiên bản cũ. Theo cách gọi sau v2.1.212, khi Agent View bật, để tạo nhánh context kiểu này trong Session hiện tại phải dùng `/subtask`; `/fork` hiện tại sẽ sao chép toàn bộ cuộc trò chuyện sang một Session nền độc lập, có thể xem, khôi phục và tiếp tục riêng. Khi tắt Agent View, `/subtask` không khả dụng, còn `/fork` vẫn giữ hành vi forked subagent cũ.

### Thời điểm sử dụng và giới hạn của hai cách sao chép

Phân tích implementation nội bộ của cộng đồng cho thấy forked subagent trong Session hiện tại sẽ tái sử dụng system prompt và message history đã được render của parent Session. Điều này có lợi cho việc tái sử dụng prompt cache, nhưng đây là quan sát implementation, không nên xem là API ổn định bên ngoài.

Mục đích chính của cách này là prompt cache.

Nếu mỗi fork child đều gọi lại logic tạo system prompt một lần, dù nội dung nhìn có vẻ giống nhau, các chi tiết như config động, danh sách tool, experimental flag vẫn có thể khiến byte không giống nhau.

Byte không giống nhau thì prompt cache hit sẽ bị ảnh hưởng.

Tái sử dụng context của parent Session vừa ảnh hưởng đến prompt cache, vừa gắn Multi-Agent với các cơ chế tầng dưới như context và đăng ký tool.

Đó cũng là lý do `/subtask` phù hợp với việc “context vừa chuẩn bị xong, lập tức bổ sung một nhánh”. Nếu nhánh cần được quản lý độc lập, tiếp tục sau đó hoặc khôi phục riêng, `/fork` hiện tại phù hợp hơn. Nếu lo các thay đổi file ảnh hưởng lẫn nhau, có thể cấu hình `isolation: "worktree"` cho Agent để đưa thay đổi vào Git Worktree độc lập.

**Agent chạy nền giải quyết vấn đề phải chờ.**

Ví dụ bạn yêu cầu một Agent chạy full code review, Agent khác phân tích log, main Session có thể tiếp tục làm design và phân tách nhiệm vụ. Khi Agent chạy nền hoàn tất, kết quả sẽ được đưa trở lại.

Nếu mở quá nhiều task nền, chi phí quản lý sẽ tăng ngay. Dùng `/tasks` để xem, tiếp quản hoặc dừng `/subtask` và các Subagent chạy nền khác trong Session hiện tại; dùng `claude agents` để mở Agent View và quản lý thống nhất các Session nền độc lập do `/fork` tạo. Cả hai đều chạy trong background, nhưng không cùng một tầng task.

![Claude Code Agent View](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/claude-agents-list-view-20260518102539932.png)

Nhưng chạy nền không có nghĩa là miễn phí. Agent chạy nền vẫn tiêu tốn token, chiếm context và trạng thái task. Mở quá nhiều thì main Session tuy không bị chặn, nhưng con người lại phải bắt đầu quản lý cả đống task.

Claude Code cũng đặt giới hạn mặc định: Claude tạo tối đa 200 Subagent trong mỗi Session thông qua tool `Agent`, mặc định chạy đồng thời tối đa 20. Có thể điều chỉnh giới hạn trước bằng `CLAUDE_CODE_MAX_SUBAGENTS_PER_SESSION`, giới hạn sau bằng `CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS`; Ultracode Session không áp dụng giới hạn chạy đồng thời mặc định.

Hai giới hạn này chủ yếu ngăn tool `Agent` tiếp tục tạo Subagent mới. `/subtask` thực hiện thủ công vẫn được tính vào quota và chiếm concurrent slot, nhưng sau khi đạt giới hạn vẫn có thể khởi chạy; `/fork` tạo Session độc lập, không tính vào quota 200 của Session hiện tại và có budget riêng.

Subagent mặc định không thể tiếp tục tạo Subagent; khi cần ủy thác lồng nhau, có thể dùng `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH` để thiết lập độ sâu. Trong Agent Teams, teammate in-process không thể tiếp tục tạo teammate, Subagent của nó cũng chỉ có thể chạy ở foreground. Đây là các biện pháp bảo vệ, không phải mục tiêu nên hướng tới; nhiệm vụ thường ngày hiếm khi cần đến.

Chi phí chung của `/subtask` và `/fork` là: cả hai đều sao chép lịch sử parent Session.

Parent Session càng sạch thì context sao chép càng có giá trị. Sau khi vừa đọc xong một module, sắp xếp xong kế hoạch nhiệm vụ, nói rõ file quan trọng và constraint, mở `/subtask` hoặc `/fork` sẽ tiết kiệm chi phí đọc lặp lại.

Ngược lại, nếu main Session đã trò chuyện rất lâu, đầy file không liên quan, phỏng đoán cũ, phương án thất bại và phán đoán tạm thời, rồi tiếp tục fork, chẳng khác nào sao chép mớ rối đó cho từng child.

Trong tình huống này, sao chép Session không phải là chia sẻ nhiệm vụ mà là sao chép sự hỗn loạn.

## Agent Teams: một nhóm instance Claude Code độc lập

### Khác biệt giữa Agent Teams và Subagent

Agent Teams là một cơ chế Multi-Agent nặng hơn trong Claude Code, đừng xem nó là bản nâng cấp của Subagent.

Một Session đóng vai trò team lead, gọi tắt là lead, chịu trách nhiệm điều phối công việc, phân nhiệm vụ và tổng hợp kết quả; các teammate làm việc độc lập, mỗi teammate có context window riêng và có thể liên lạc với nhau.

Điểm này rất dễ nhầm: teammate không kế thừa lịch sử trò chuyện của lead. Nó giống một Session Claude Code mới mở, sẽ load `CLAUDE.md`, MCP servers và Skills của project hiện tại, đồng thời nhận spawn prompt do lead gửi, nhưng những trao đổi qua lại, phỏng đoán tạm thời và phương án bị bác bỏ trước đó sẽ không tự động được mang sang.

Vì vậy spawn prompt không thể chỉ viết “bạn xem backend nhé”. Phải viết rõ đường dẫn quan trọng, giới hạn đã biết và output mong muốn. Nếu không, teammate nhận được một window sạch, nhưng cũng có thể sạch đến mức không biết vừa rồi bạn đã thảo luận gì.

Subagent thường là “làm xong rồi quay lại báo cáo”. Teammate thì cộng tác thông qua shared task list và mailbox: người nhận nhiệm vụ, người bổ sung thông tin, cuối cùng lead tổng hợp.

Teammate cũng có thể tái sử dụng definition Subagent có sẵn. Ví dụ bạn đã viết `security-reviewer`, khi spawn teammate có thể chỉ định agent type này. Nó sẽ dùng lại `tools` và `model` trong definition đó, đồng thời nối nội dung definition vào system prompt của teammate. Lưu ý, hai field frontmatter `skills` và `mcpServers` sẽ không có hiệu lực qua đường này; teammate vẫn load Skills / MCP servers theo config project và user của Session thông thường.

Trước khi sử dụng còn phải bật experimental flag. Hiện Agent Teams vẫn là experimental và mặc định tắt, cần thiết lập:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Hoặc thiết lập environment variable cùng tên trong shell.

### shared task list, mailbox và teammate mode

Agent Teams không biến vài teammate thành những người cùng spam message trong một khung chat lớn. Nó chủ yếu dựa vào hai thứ để cộng tác:

| Component        | Tác dụng                                                       |
| ---------------- | -------------------------------------------------------------- |
| shared task list | Ghi lại task của team, teammate có thể nhận và hoàn thành task |
| mailbox          | Teammate gửi message, yêu cầu thông tin và đồng bộ trạng thái  |

Điểm mới không chỉ là báo cáo kết quả. Agent Teams duy trì shared task list và mailbox, để teammate có thể nhận task, đồng bộ trạng thái và bổ sung thông tin cho nhau.

Xem shared task list như TODO thông thường sẽ đánh giá thấp cơ chế này. Task có ba trạng thái `pending`, `in progress`, `completed`, cũng có thể thiết lập dependency; khi dependency chưa hoàn tất, task phía sau không thể được nhận. Khi nhiều teammate cùng tranh một task, Claude Code dùng file lock để tránh conflict khi nhận đồng thời.

Phần message cũng vậy. Lead đặt tên cho mỗi teammate, sau đó có thể gửi message theo tên. Khi teammate rảnh hoặc thất bại, nó cũng tự động thông báo cho lead, không cần lead liên tục polling.

Trong phân tích source code có thể thấy implementation tầng thấp hơn: mailbox là inbox dạng file, khi ghi sẽ tính đến lock đồng thời; task list giúp teammate không chỉ nhận prompt mà còn có thể claim work item.

Subagent thông thường giống ủy thác một lần. Agent Teams duy trì task và message dùng chung, có cảm giác giống một work queue nhỏ.

Cách viết prompt cũng thay đổi theo. Khi dùng Subagent, tốt nhất nói rõ nhiệm vụ trong một lần; khi dùng Agent Teams, lead có thể tách nhiệm vụ lớn vào shared task list trước, rồi teammate tiếp tục tiến lên xoay quanh task list và message.

![Claude Code Agent Teams: nhiều teammate phối hợp review xoay quanh một luồng phân tích hoàn chỉnh](https://oss.javaguide.cn/github/javaguide/ai/coding/claude-code-agent-teams-agentinvest-review.png)

Hình trên là một ví dụ gần với project thực tế hơn: team lead trước tiên tách luồng phân tích hoàn chỉnh của AgentInvest thành bốn hướng: backend SSE, orchestration của Agent, render frontend, test và rủi ro resilience, rồi spawn 4 teammate lần lượt nhận nhiệm vụ. Trọng tâm không phải mở thêm vài task tìm kiếm, mà là teammate phân công và tiến hành xoay quanh cùng một shared task list, cuối cùng lead tổng hợp các vấn đề xuyên module.

`--teammate-mode` dùng để điều khiển cách teammate hiển thị:

| Mode         | Ý nghĩa                                                                          |
| ------------ | -------------------------------------------------------------------------------- |
| `in-process` | Chế độ mặc định, hiển thị teammates trong process hiện tại                       |
| `auto`       | Dùng split pane khi tmux / iTerm2 khả dụng, nếu không thì fallback về in-process |
| `tmux`       | Dùng split pane của tmux hoặc iTerm2                                             |
| `iterm2`     | Dùng native split panes của iTerm2, thêm từ v2.1.186                             |

Giá trị mặc định của `teammateMode` được đổi từ `auto` sang `in-process` ở v2.1.179.

Cần chú ý những thay đổi phiên bản kiểu này khi viết script và tài liệu team. Nhiều tutorial trên mạng vẫn mặc định khuyên dùng tmux hoặc vẫn dùng quy trình tạo team cũ; sao chép nguyên xi dễ không khớp phiên bản hiện tại.

### Thay đổi phiên bản sau v2.1.178

Trong implementation cũ có thể thấy các chi tiết như `TeamCreate` / `TeamDelete`, `team file`, `team_name`. Những nội dung này hữu ích để hiểu quá trình tiến hóa của Agent Teams, nhưng không thể viết trực tiếp thành cách dùng ổn định hiện tại.

Sau v2.1.178, khi bật `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`, lúc teammate đầu tiên được tạo, Agent Team của Session hiện tại sẽ tự động được hình thành, không còn cần bước tạo riêng. Một Session chỉ có một Team tại một thời điểm, không thể tạo thêm Team có tên khác.

Tool `TeamCreate` và `TeamDelete` đã bị loại bỏ, config Team runtime sẽ tự động được dọn khi Session kết thúc.

Parameter `team_name` trong tool `Agent` vẫn được chấp nhận nhưng sẽ bị bỏ qua. `team_name` trong hook payload của `TaskCreated`, `TaskCompleted`, `TeammateIdle` cũng là field tương thích.

Trong thời gian Agent Team chạy, nó dùng hai loại directory local: Team runtime config ở `~/.claude/teams/{team-name}/config.json`, Task list ở `~/.claude/tasks/{team-name}/`.

Hai directory này được Claude Code tự động tạo và cập nhật. Config chứa thông tin runtime như Session ID, Pane ID, Members, sẽ bị xóa sau khi Session kết thúc; Task list được giữ local, sau khi khôi phục Session vẫn có thể dùng tiếp, chu kỳ dọn dẹp do `cleanupPeriodDays` điều khiển. Không chỉnh thủ công các file này, cũng không viết `.claude/teams/teams.json` trong project và mong nó có hiệu lực.

Vì vậy khi đọc source code cũ, mình sẽ tách riêng:

- Implementation cũ giúp hiểu vì sao Agent Teams có task list, mailbox và directory team;
- Cách sử dụng hiện tại phải lấy tài liệu chính thức làm chuẩn, không tiếp tục hướng dẫn user gọi `TeamCreate`.

## Permission, chi phí và thay đổi phiên bản

**Yêu cầu permission trước tiên quay về lead**

Điều đáng lo nhất của Multi-Agent là worker vượt qua permission của main Session để tự sửa file hoặc chạy command.

Claude Code không để teammate tự quyết. Yêu cầu permission cần user xác nhận vẫn sẽ quay về lead.

Trong phân tích source code có thể thấy leader permission bridge: nếu teammate in-process cần user xác nhận, trước tiên nó sẽ đưa request vào `ToolUseConfirmQueue` của leader, UI có worker identifier. Khi bridge không khả dụng, nó mới fallback sang đường mailbox.

User vẫn phán đoán permission ở một nơi, không cần theo dõi riêng popup xác nhận trong nhiều teammate.

Subagent cũng có thể cấu hình phạm vi tool và hooks riêng.

Tài liệu Subagent chính thức từng đưa ví dụ chỉ cho phép query database chỉ đọc: dùng hook `PreToolUse` kiểm tra command Bash, nếu phát hiện thao tác ghi như `INSERT`, `UPDATE`, `DELETE` thì thoát và chặn thực thi.

Thiết kế này cùng hướng với phân tầng an toàn khi gọi tool: thao tác rủi ro thấp có thể nới lỏng, thao tác rủi ro cao cần xác nhận; khi liên quan đến xóa file, commit, deploy, ghi database thì không thể chỉ dựa vào một câu prompt để ràng buộc.

![Phân tầng rủi ro an toàn khi gọi tool: ghép chiến lược kiểm soát theo mức rủi ro](https://oss.javaguide.cn/github/javaguide/ai/llm/structured-output-function-calling-tool-call-security.png)

**Chi phí chủ yếu nằm ở nhiều context độc lập**

Agent Teams đắt, chủ yếu vì mỗi teammate là một instance Claude Code độc lập.

Mình rút gọn việc kiểm soát chi phí thành vài thói quen sử dụng:

- Phải chỉ định rõ model của teammate, hoặc thiết lập `Default teammate model` trong `/config`; nó không nhất thiết tự đi theo `/model` của lead;
- Hầu hết workflow nên bắt đầu với 3-5 teammate, ba teammate tập trung thường hữu ích hơn năm teammate phân tán;
- Đừng tách task quá vụn, cũng đừng để task lớn đến mức không check-in trong thời gian dài; tốt nhất là một function, một file test hoặc một lượt review, những deliverable tự chứa;
- Người mới nên bắt đầu bằng các task không viết code như nghiên cứu, review, điều tra bug;
- Nếu cần sửa code song song, cố gắng để mỗi teammate phụ trách file khác nhau, tránh hai teammate cùng sửa một file.

Nói cho cùng, đây vẫn là quản lý context.

Mở ba teammate tương đương đồng thời duy trì nhiều window, không phải chia một window thành ba phần. Khi task thực sự có thể chạy song song, chi phí này đáng; khi task phụ thuộc chặt chẽ và phải liên tục chờ kết quả của nhau, chưa chắc đã đáng, cuối cùng rất dễ biến thành tự tăng việc cho mình.

**Thông tin phiên bản chỉ nên xem như snapshot**

Bài viết được đối chiếu theo Claude Code v2.1.218 (2026-07-24). Thông tin phiên bản sẽ thay đổi, tính năng cụ thể vẫn phải lấy tài liệu chính thức và changelog làm chuẩn. Đặc biệt với các tính năng thay đổi nhanh như Agent Teams, Subagent, Skills, command và tên tool trong bài viết cũ không nhất thiết còn hiệu lực.

## Chọn thế nào khi sử dụng thực tế?

**Nhiệm vụ nhỏ dùng một Agent**

Nhiệm vụ rõ ràng, phạm vi thay đổi nhỏ, context không phức tạp thì dùng một Agent, không cần nghĩ quá nhiều.

Ví dụ:

- Sửa một function;
- Bổ sung một unit test;
- Điều chỉnh một config;
- Giải thích một đoạn code.

Với loại nhiệm vụ này, vừa bắt đầu đã mở Subagent hoặc Agent Teams chỉ làm tăng chi phí điều phối. Để một Session hoàn thành thay đổi nhỏ sẽ ổn định hơn.

**Nhiệm vụ ở nhánh dùng Subagent**

Mình xem Subagent như người “đi xử lý một chuyến”.

Ví dụ yêu cầu nó review chỉ đọc một module, tìm một nhóm error log, làm rõ context của một module, hoặc giúp xác minh độc lập code vừa sửa.

Các việc này có một điểm chung: quá trình không nhất thiết quan trọng, kết luận mới quan trọng. Main Session chỉ cần biết vấn đề nằm ở đâu, bằng chứng ở đâu, bước tiếp theo sửa thế nào; không cần dồn mọi kết quả tìm kiếm, phỏng đoán tạm thời và đường đi thất bại vào đó.

Khi thực sự phải động đến code cốt lõi, mình vẫn thích ở lại main Session để làm hơn. Subagent chịu trách nhiệm tìm manh mối và xác minh kết quả, main Session chịu trách nhiệm phán đoán và triển khai.

**Cần cộng tác mới dùng Agent Teams**

Mình thận trọng hơn khi dùng Agent Teams. Nó phù hợp với những nhiệm vụ mà chỉ “kiểm tra xong rồi báo cáo” là chưa đủ.

Ví dụ một tính năng mới đồng thời liên quan đến API backend, tương tác frontend, chiến lược test và review ngược. Các teammate không chỉ mỗi người xem một phần, mà còn phải hỏi nhau: field API này đã đổi, frontend có cần theo không? Có cần bổ sung test không? Hiện ai đang rảnh để nhận phần tiếp theo?

Lúc này shared task list và cơ chế message mới có giá trị. Nếu không thì chỉ là mở thêm vài worker, mỗi worker chạy xong một đoạn rồi quay về tổng kết; dùng Subagent là đủ.

Đừng cố tách các tình huống sau: thay đổi liên tiếp trong cùng một file, nhiệm vụ phụ thuộc thứ tự chặt chẽ, nhiệm vụ cần một người liên tục nắm toàn bộ context. Cố tách sẽ chỉ làm tăng chi phí phối hợp.

Nếu nhiều Agent đều cần sửa code, tốt nhất hãy tách workspace trước. Cách ổn định hơn là một Agent một Git Worktree, mỗi branch chỉ chứa một nhiệm vụ rõ ràng, cuối cùng do người hoặc lead merge và nghiệm thu.

![Claude Code Git Worktree](https://oss.javaguide.cn/github/javaguide/ai/coding/claude-code-git-worktree.png)

**Phân tách rõ nhiệm vụ trước, rồi mới tăng số Agent**

Khi ranh giới nhiệm vụ không rõ, nhiều Agent chỉ tạo ra nhiều kết quả trung gian thiếu nhất quán hơn. Trước tiên dùng một Agent để làm rõ mục tiêu và dependency; việc có thể xác minh độc lập thì giao cho Subagent; khi context hiện tại đáng tái sử dụng thì chọn `/subtask` hoặc `/fork`; chỉ khi thực sự cần liên lạc giữa các vai trò mới bật Agent Teams.

Cụ thể hơn, trước tiên có thể chạy thành pipeline tuần tự: Plan đưa ra phương án chỉ đọc, Code làm một task, Test bổ sung xác minh, Review chỉ xem diff. Sau khi workflow này ổn định, mới tách các khâu có thể thực hiện độc lập cho các Agent khác nhau.

![Pipeline cộng tác ba Agent Multi-Agent](https://oss.javaguide.cn/github/javaguide/ai/coding/spec-coding-multi-agent-pipeline.png)

## Tổng kết

Viết đến đây, quay lại câu hỏi ở đầu bài: nên chọn Subagent, Fork và Agent Teams trong Claude Code thế nào?

Đừng trước tiên chăm chăm vào việc “nhiều Agent có nhanh hơn không”. Mình thích xem đây là một cách quản trị context hơn: main Session chịu trách nhiệm phán đoán, lập kế hoạch và triển khai; những việc ở nhánh như tìm kiếm, review, xác minh dễ làm bẩn context thì có thể tách được là tách.

Subagent phù hợp để cô lập quá trình. Để nó tự đọc file, xem log, review chỉ đọc; main Session chỉ nhận kết luận và bằng chứng.

`/subtask` phù hợp để tái sử dụng context đã được sắp xếp trong Session hiện tại, hoàn thành một nhánh và trả kết quả; khi Agent View bật, `/fork` phù hợp để sao chép toàn bộ cuộc trò chuyện thành một Session nền độc lập để quản lý riêng sau đó. Khi main Session đã rất lộn xộn, cả hai chỉ sao chép sự hỗn loạn.

Agent Teams nặng hơn một tầng. Chỉ nên dùng khi nhiệm vụ thực sự cần nhiều teammate nhận task, liên lạc với nhau và dùng chung task list. Nó tiêu tốn chi phí của nhiều context độc lập, đồng thời mang đến chi phí phối hợp.

Thứ tự sử dụng của mình là: nhiệm vụ nhỏ dùng một Agent; nhánh sạch dùng Subagent thông thường; khi cần tái sử dụng context thì lựa chọn giữa `/subtask` và `/fork`; khi thực sự cộng tác xuyên module mới mở Agent Teams. Giá trị chính của nó là cô lập quá trình và làm rõ trách nhiệm, còn chạy song song chỉ là kết quả sau khi nhiệm vụ có thể được tách độc lập.

Đọc thêm [AIGuide: Hướng dẫn phát triển ứng dụng AI, thực chiến AI Coding và phỏng vấn](https://mp.weixin.qq.com/s/le3RzJsaAH22auUoB05y1Q), [Hướng dẫn thực chiến context engineering](https://javaguide.cn/ai/agent/context-engineering.html) và [Spec Coding: Lập trình theo đặc tả](https://javaguide.cn/ai-coding/practices/spec-coding.html); bài đầu thiên về cô lập context, bài sau thiên về pipeline cộng tác nhiều Agent.

## Tài liệu tham khảo

- [Claude Code Commands](https://code.claude.com/docs/en/commands)
- [Create custom subagents](https://code.claude.com/docs/en/sub-agents)
- [Run agents in parallel](https://code.claude.com/docs/en/agents)
- [Orchestrate teams of Claude Code sessions](https://code.claude.com/docs/en/agent-teams)
- [Claude Code Changelog](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Claude Code Source Code Deep Research Report (phân tích source code cộng đồng, không chính thức)](https://claudeai.dev/docs/mechanics/development/claude-code-source-deep-research/)
