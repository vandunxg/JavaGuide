---
title: "Loop Engineering là gì? Vì sao nói đây là rượu cũ bình mới?"
description: "Bắt đầu từ Agent Loop, Context Engineering, Harness, Skills, MCP, Sub-agent và Claude Code /loop, /goal, bài viết giải thích Loop Engineering thực sự giải quyết vấn đề gì và khi nào đáng dùng."
category: Phát triển ứng dụng AI
head:
  - - meta
    - name: keywords
      content: Loop Engineering,Agent Loop,AI Agent,Claude Code,/loop,/goal,Context Engineering,Harness Engineering,Agent Skills,MCP,Lập trình AI
---

Sau khi CI thất bại, Agent có thể đọc log lỗi, định vị file liên quan, chạy tập test tối thiểu rồi ghi kết quả điều tra vào Issue. Khi lần điều tra đầu tiên chưa đi đến kết luận, nhiệm vụ còn gặp một vấn đề thực tế hơn: ai sẽ khởi động vòng tiếp theo, tiếp tục đọc tài liệu nào và khi nào nên dừng để giao cho con người xử lý?

Loop Engineering bàn về chính quy trình bên ngoài này. Nó kết nối từng lần thực thi của Agent: CI, PR hoặc tác vụ định kỳ kích hoạt vòng tiếp theo, rule của project và evidence liên quan được đưa vào context, test và review quyết định kết quả có đáng tin hay không, còn state record cho phép tác vụ sau tiếp tục từ nơi lần trước dừng lại.

Theo các thảo luận công khai, tên gọi này bắt đầu phổ biến vào đầu tháng 6 năm **2026**, Addy Osmani đăng bài liên quan vào ngày 7 tháng 6. Các khả năng `/loop`, `/goal`, Automations trong Claude Code và Codex, kết hợp với Skills, Sub-agent, workspace isolation và MCP/Connector, đã có thể tạo thành một flow tương tự.

## Loop Engineering thực sự là gì?

Loop Engineering dùng để sắp xếp các tác vụ nhiều vòng của Agent. Nó đưa cách trigger task, tài liệu và action có thể thực hiện trong mỗi vòng, signal nghiệm thu, vị trí lưu state, cùng điều kiện dừng hoặc human takeover vào cùng một flow.

Lấy module `auth` làm ví dụ, `goal` có thể viết là “tất cả test đều pass, thử tối đa 5 vòng”. Cùng một task config còn ghi lại nguồn trigger, file và rule có thể truy cập, phạm vi được phép sửa, cùng vị trí ghi kết quả.

Giả sử CI trigger task này: trước tiên Agent đọc rule của project và log lỗi, sau khi định vị file liên quan thì chạy target test. Test output, lint, type check, screenshot hoặc review comment sẽ quay lại vòng tiếp theo, giúp quyết định có tiếp tục hay không. Các phương án đã thử, nguyên nhân thất bại và bước tiếp theo được ghi vào file bên ngoài, Issue, Linear card hoặc database; khi đạt giới hạn số vòng hoặc budget, thiếu permission hay cần business judgment, flow sẽ dừng và giao cho con người xử lý.

![Vòng lặp bên ngoài của Loop Engineering](https://oss.javaguide.cn/github/javaguide/ai/agent/loop-engineering-outer-loop.webp)

Prompt, context và tool description vẫn quyết định một model call được thực thi thế nào. Loop bổ sung scheduling task, chuẩn bị tài liệu, xác minh kết quả và khôi phục state trước và sau call, để vòng tiếp theo có thể tiếp tục xử lý từ vòng trước.

## Nó thực ra mượn những concept cũ nào?

### Agent Loop / ReAct: vòng lặp bên trong đã tồn tại từ lâu

![Flow hoạt động của Agent Loop](https://oss.javaguide.cn/github/javaguide/ai/agent/agent-loop-flow.png)

Trình tự cơ bản của Agent Loop không thay đổi: đọc context hiện tại, giao cho LLM quyết định bước tiếp theo, gọi tool hoặc tạo kết quả, rồi ghi tool output trở lại context; sau khi đạt điều kiện dừng thì kết thúc.

ReAct cũng theo ý tưởng này: Reasoning và Acting luân phiên, model vừa đi vừa quan sát, sau khi nhận feedback bên ngoài thì quyết định bước tiếp theo.

[Khái niệm nền tảng của AI Agent](https://javaguide.cn/ai/agent/agent-basis.html) đã giới thiệu vòng lặp này. Troubleshooting sự cố production, đọc codebase và định vị test failure đều không có path hoàn chỉnh được xác định trước; model cần điều chỉnh bước tiếp theo dựa trên evidence nhận được mỗi lần.

Cần phân biệt hai tầng loop. Agent Loop xảy ra trong một lần thực thi task: model reasoning, gọi tool, đọc result rồi quyết định bước tiếp theo. Outer Loop hoạt động sau khi task này kết thúc, chẳng hạn chờ CI event tiếp theo, kiểm tra test result, lưu record điều tra rồi quyết định có khởi động lại Agent hay không.

| Tầng                   | Ai lặp                                          | Mỗi vòng làm gì                                           | Điều kiện dừng điển hình                             |
| ---------------------- | ----------------------------------------------- | --------------------------------------------------------- | ---------------------------------------------------- |
| Inner Agent Loop       | Agent tự thực hiện                              | Suy nghĩ, gọi tool, quan sát result, tiếp tục bước sau    | Không còn cần tool, trả về kết quả cuối cùng         |
| Outer Engineering Loop | Hệ thống scheduling hoặc flow do con người viết | Đánh thức Agent, phân bổ task, xác minh result, ghi state | Đạt goal, vượt budget, thất bại thì chuyển cho người |

### Workflow / Graph / Loop: controlled back edge đã tồn tại từ lâu

Trong workflow graph, Loop thường được biểu diễn bằng back edge.

Back edge là directed edge đi từ node phía sau về node phía trước: flow đã đi đến node “review”, nhưng vì một điều kiện không đạt nên đi theo edge này trở về node “sửa”, rồi thực hiện lại các bước tiếp theo.

![Tổng quan quan hệ giữa Workflow, Graph và Loop](https://oss.javaguide.cn/github/javaguide/ai/workflow/workflow-graph-loop-relation.svg)

Trong “tạo draft đầu tiên → review → không đạt thì sửa → review lại”, conditional edge khi review không đạt chính là back edge từ “review” về “sửa”; khi review đạt thì rời khỏi loop. [Workflow, Graph và Loop trong AI Workflow](https://javaguide.cn/ai/agent/workflow-graph-loop.html) giải thích đầy đủ hơn về cấu trúc này. Runtime config còn phải ghi rõ số vòng tối đa, timeout, Token budget và cách downgrade sau failure, để back edge không bị thiếu lối ra.

Code Agent kéo dài cùng back edge đó đến Claude Code, Codex, CI, GitHub, Issue system và local repository: sau khi test failure thì đọc error, sửa file, chạy lại command, rồi để external signal quyết định có tiếp tục hay không.

### Context Engineering: mỗi vòng nên cho Agent xem gì

Khi điều tra một CI failure đến vòng thứ ba mà vẫn chưa đi đến kết luận, raw log, test output, change record và các phán đoán mâu thuẫn rất dễ chồng chất. Nếu nhét tất cả vào context, rule của project lại dễ bị nhấn chìm, còn các phương án đã loại trừ có thể bị chạy lại.

![Sự khác nhau giữa Context Engineering và Prompt Engineering](https://oss.javaguide.cn/github/javaguide/ai/context-engineering/context-engineering-vs-context-engineering-dimension-comparison.png)

Trước mỗi call, trước hết nên đưa các rule thường trực như `AGENTS.md`, `CLAUDE.md` và coding convention vào, sau đó tải file liên quan, test output, mô tả Issue hoặc design document theo failure hiện tại. Các entry point troubleshooting như traceId, error code, log path giữ nguyên giá trị; process đã verify được nén thành conclusion và ghi vào external state.

Khi window gần đạt giới hạn, cần cân nhắc giữa nén history, dọn tool result cũ và ghi progress xuống storage. Nhờ vậy vòng tiếp theo có thể nối thẳng vào evidence, tránh đọc lại project, đoán rule hoặc giải thích lại cùng một error.

### Harness Engineering: execution environment bên ngoài model

Trong [Harness Engineering](https://javaguide.cn/ai/agent/harness-engineering.html), Agent có thể được tách thành Model + Harness. Model phụ trách reasoning và generation, còn Harness cung cấp environment, tool, feedback, sandbox, permission, observation và recovery.

![Quan hệ giữa Harness và Prompt/Context Engineering](https://oss.javaguide.cn/github/javaguide/ai/harness/harness-engineering-layers-arch.png)

Có thể xem Harness là runtime environment của một single-round task. CI triage có thể đọc log nào, có được sửa file không, có thể chạy test command nào đều do Harness quyết định; Loop lại quyết định khi nào khởi động environment này, lưu result ở đâu và có cần giao cho Agent khác kiểm tra hay không. Chỉ có goal mà không có file permission, verification command và failure handling thì task vẫn không thể chạy unattended.

### Skills: ghi lại kinh nghiệm phải giải thích lặp lại trong mỗi vòng

![Execution chain của Agent](https://oss.javaguide.cn/github/javaguide/ai/skills/skills-agent-execution-link.png)

Khi CI troubleshooting lặp đi lặp lại, test command của repository, directory cấm sửa, yêu cầu format, PR template và rule xác nhận database migration không nên được giải thích lại ở mỗi vòng.

Những nội dung này có thể ghi vào Skill: `description` match với task CI troubleshooting, còn `SKILL.md` lưu directory được phép đọc, test command, PR template và rule xác nhận migration. Khi task match, nội dung chính được load; failure lần sau vẫn dùng cùng bộ giới hạn.

![Progressive Disclosure của Skill](https://oss.javaguide.cn/github/javaguide/ai/skills/agent-skills-progressive-disclosure.webp)

Skill ghi lại operation instruction được dùng lặp lại trong project, không phải một đoạn Prompt tạm thời ghép cho conversation hiện tại.

### MCP: giúp Loop tiếp cận tool thực tế

GitHub Actions, log platform, Linear và Slack đều có API riêng. CI troubleshooting, PR babysit và task triage phải chuyển qua lại giữa các system này; nếu tích hợp tất cả riêng lẻ, trước mặt Agent sẽ xuất hiện nhiều bộ tool description và cách gọi khác nhau.

![Sơ đồ MCP](https://oss.javaguide.cn/github/javaguide/ai/skills/mcp-simple-diagram.png)

MCP Server cung cấp GitHub, Issue system, log platform, internal document và database dưới dạng discoverable tool; Agent Runtime phụ trách lựa chọn, còn business system vẫn thực hiện permission và data validation.

Tool permission cần được cấu hình theo side effect. Khi unattended Loop có write permission quá lớn, nó có thể sửa nhầm data, gửi nhầm message, gọi lặp API đắt tiền hoặc bị prompt injection dụ đọc file không liên quan; permission, audit, rate limiting, data masking và human confirmation nên được deploy cùng MCP tool.

## Vậy nó mới ở đâu?

TDD, CI, ReAct và workflow graph đã có loop từ lâu. Code Agent kết nối các action bên ngoài vốn trước đây do con người thực hiện: đọc error, định vị file, chạy lại command, cập nhật việc cần làm rồi đưa result về vòng tiếp theo.

Lấy test failure làm ví dụ, scheduled task hoặc CI event có thể tạo worktree độc lập, load Skill của project, để implementation Agent sửa, verification Agent kiểm tra, rồi ghi result vào external state. Khi test vẫn failure, vòng tiếp theo tiếp tục dựa trên log và test result; khi thiếu permission, không có tiến triển hoặc gặp operation rủi ro cao, flow giao cho con người.

Vì vậy, trọng tâm của Loop Engineering thực ra chỉ có ba điểm: **vòng tiếp theo cần evidence nào để tiếp tục, action nào bắt buộc phải tạm dừng, và state của vòng trước được lưu ở đâu.**

![Vòng lặp bên ngoài của Loop Engineering](https://oss.javaguide.cn/github/javaguide/ai/agent/loop-engineering-outer-loop.webp)

## Có thể hiểu /loop, /goal của Claude Code thế nào?

`/loop` chạy lại Prompt theo thời gian, còn `/goal` quyết định có tiếp tục hay không dựa trên completion condition. Có thể tham khảo thêm [Giải thích chi tiết các command của Claude Code](https://javaguide.cn/ai-coding/claudecode-commands.html).

![Khuyến nghị dùng command loop của Claude Code](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/claudecode-father-loop.png)

`/loop` chủ yếu giải quyết vấn đề “một lúc sau xem lại”. Nó lặp lại một prompt trong session hiện tại. Bạn có thể đặt interval cố định, chẳng hạn kiểm tra deployment mỗi 5 phút; cũng có thể không đặt interval để Claude tự chọn thời gian chờ cho lần tiếp theo dựa trên observation.

`/loop` không có argument sẽ chạy maintenance prompt tích hợp sẵn, hoặc đọc `.claude/loop.md`, `~/.claude/loop.md` làm prompt mặc định.

```bash
/loop 5m "Kiểm tra deployment đã hoàn tất chưa và báo cáo trạng thái hiện tại"
/loop 30m /code-review
/loop "Kiểm tra CI và review comments của PR, có thay đổi thì xử lý, không có thay đổi thì hoãn lại"
```

`/goal` chủ yếu giải quyết vấn đề “goal này đã hoàn thành chưa”. Bạn đưa cho nó một completion condition có thể verify, Claude sẽ thúc đẩy từng vòng; sau mỗi vòng, một model nhỏ độc lập dựa trên evidence đã xuất hiện trong conversation để quyết định condition đã đạt hay chưa. Chưa đạt thì tiếp tục, đạt thì dừng.

```bash
/goal tất cả unit test của module auth đều pass, và npm test -- tests/auth có exit code bằng 0; tối đa 5 vòng, nếu nguyên nhân failure giống nhau trong 2 lần liên tiếp thì dừng và báo cáo
/goal hoàn tất migration các component trong src/legacy, npm run build pass, và git diff chỉ chứa src/legacy cùng các file test tương ứng
```

Có thể ghi nhớ hai câu này: `/loop` quyết định khi nào thức dậy lần tiếp theo, `/goal` phán đoán khi nào được xem là hoàn tất.

Stop hook hoặc Agent SDK có thể giao continuation condition cho script, Prompt hoặc external evaluator, đồng thời thực hiện deterministic check, permission interception và state persistence sau mỗi vòng.

“Tiếp tục sửa cho đến khi test pass, thử tối đa 5 lần” do `/goal` kiểm tra completion condition. Deployment polling, PR babysit, build dài và code review định kỳ do `/loop` quan sát lại theo lịch.

`/loop` là temporary scheduling theo session-scoped: task chỉ trigger khi Claude Code đang chạy và idle; đóng terminal, thoát session hoặc mở session mới đều ảnh hưởng đến nó; `--resume` hoặc `--continue` chỉ khôi phục task chưa hết hạn; loop task tự động hết hạn tối đa sau 7 ngày.

Khi task phải chạy ổn định lâu dài, xuyên máy và xuyên restart, vẫn nên cân nhắc Routines, Desktop scheduled tasks, GitHub Actions, CI/CD hoặc task scheduling system riêng.

Trước khi chạy `/loop`, cần thu hẹp permission, viết rõ mục tiêu polling và stop condition; trước khi chạy `/goal`, cần viết completion condition thành result có thể verify và yêu cầu Claude hiển thị test, build hoặc diff check result. Với critical path, hãy commit trước rồi mới để Agent sửa, để khi lỗi có thể quay về một commit point rõ ràng.

## Loop có thể chia thành những loại nào?

Khác biệt của Loop chủ yếu nằm ở “ai trigger vòng tiếp theo”. `/loop` và `/goal` lần lượt đại diện cho việc đánh thức theo thời gian và tiếp tục theo completion condition; CI event và human approval cũng có thể là trigger point. Bảng dưới đây phân loại theo dimension này.

| Loại                | Cách trigger                                           | Task phù hợp                                      | Tool đại diện                                 |
| ------------------- | ------------------------------------------------------ | ------------------------------------------------- | --------------------------------------------- |
| Time-driven Loop    | Mỗi N phút, mỗi ngày, mỗi tuần                         | PR babysit, CI check, log inspection              | `/loop`、Codex Automations、cron              |
| Event-driven Loop   | CI failure, tạo Issue, update PR                       | Failure triage, xử lý comment, tóm tắt alert      | GitHub Actions, Webhook, Claude Code Channels |
| Goal-driven Loop    | Sau khi vòng trước kết thúc, kiểm tra goal đã đạt chưa | Sửa test, migration API, bổ sung coverage         | `/goal`, Stop hook, Agent SDK                 |
| Human approval Loop | Dừng lại để xác nhận trước action quan trọng           | Thay đổi rủi ro cao, release, thay đổi permission | approval gate, draft PR, review queue         |

Bảng này cũng giải thích câu “rượu cũ bình mới” ở trên. Các action engineering như trigger, scheduling, verification và approval đều không mới, chỉ là hiện nay được đặt lại quanh code Agent.

## Một Loop có thể triển khai trông như thế nào?

Lấy “tự động xử lý CI failure mỗi ngày” làm ví dụ. Flow ở đây được sắp xếp theo quy trình troubleshooting thường gặp, không tương ứng với một case thực tế hoàn chỉnh được công khai của công ty nào. Version đầu tiên chỉ làm triage, không tự động sửa code và không tự động merge.

Version đầu tiên chỉ verify ba việc: Agent có tìm được evidence đúng không, có phân biệt được fact và guess không, có ghi state theo format thống nhất không. Chỉ sau khi cả ba ổn định mới cân nhắc mở permission sửa lỗi rủi ro thấp.

CI triage có thể được trigger lúc 9 giờ sáng mỗi ngày hoặc khi CI failure, đọc failure gần nhất, PR liên quan, commit gần nhất và log của test failure. Nó load `AGENTS.md` của project cùng Skill `ci-triage`, chỉ đọc file của module liên quan, đồng thời phân biệt environment flakiness, test không ổn định, code regression và dependency issue.

Khi có thể reproduce ở local thì chạy tập test tối thiểu; không reproduce được thì giữ lại evidence. Conclusion được ghi vào `TODO.md`, GitHub Issue hoặc Linear card, rồi đánh dấu “có thể tự động sửa”, “cần owner xác nhận” hoặc “nghi là lỗi ngẫu nhiên”. Flow không trực tiếp push code, không sửa production config và không retry liên tiếp quá 3 lần.

![Ví dụ CI troubleshooting Loop](https://oss.javaguide.cn/github/javaguide/ai/agent/loop-engineering-ci-triage-loop.webp)

Sau khi version này ổn định, mới từng bước thêm automatic repair:

- Với các vấn đề rủi ro thấp như “dependency version conflict”, “formatting failure”, “cập nhật test snapshot rõ ràng”, có thể mở worktree độc lập để Agent thử sửa.
- Sau khi sửa xong bắt buộc chạy target test.
- Nếu pass thì chỉ tạo PR, không tự động merge.
- Một Agent review khác kiểm tra lần hai dựa trên Skill của project và diff.
- Khi failure hoặc không chắc chắn thì quay lại human queue.

Trong Loop này có thể thấy các component đã nhắc ở trên:

| Component           | Làm gì trong ví dụ này                                                    |
| ------------------- | ------------------------------------------------------------------------- |
| Automation          | Khởi động mỗi ngày hoặc khi CI failure                                    |
| Skill               | Cố định quy trình CI troubleshooting, test command và rule của repository |
| MCP / Connector     | Đọc GitHub, CI, Issue và log platform                                     |
| Context Engineering | Chỉ load log, file và rule liên quan đến failure                          |
| Worktree            | Cô lập nhánh automatic repair, tránh làm bẩn main workspace               |
| Sub-agent           | Một agent phụ trách implementation, một agent phụ trách verification      |
| Memory / State      | Ghi lại phương án đã thử, nguyên nhân failure và bước tiếp theo           |
| Stop Condition      | Test pass, đạt retry limit hoặc gặp operation rủi ro cao                  |

“Chạy lúc 9 giờ mỗi ngày” chỉ cung cấp thời điểm khởi động. CI link và log failure xác định entry point troubleshooting, command reproduce tối thiểu và test result quyết định có tiếp tục hay không, còn PR diff và trạng thái human confirmation quyết định change có thể đi sang bước tiếp theo hay không. Không có các external evidence này, scheduled task chỉ có thể gửi lại cùng một Prompt.

## Scenario nào đáng làm Loop?

Loop phù hợp với các task cần được thực hiện lặp lại và có acceptance signal có thể kiểm tra. Log, exit code, coverage và diff đều có thể làm căn cứ tiếp tục hoặc dừng. Các task sau thường có thể viết được loại condition này:

- CI failure troubleshooting ban đầu: có log, có test result và có failure signal rõ ràng.
- Thay đổi dependency version: sửa trong branch độc lập, nghiệm thu bằng test exit code và build result.
- Bổ sung test coverage: goal có thể định lượng, chẳng hạn coverage của một module tăng từ 62% lên 75%.
- Đồng bộ tài liệu: cập nhật user document hoặc API document dựa trên diff gần nhất, cuối cùng qua human review.
- Migration cơ học quy mô lớn: chẳng hạn CommonJS sang ESM, thay API của component cũ hoặc sửa format.
- PR / Issue triage: đọc thông tin, phân loại, bổ sung summary và đánh dấu priority.

Các task sau khó nghiệm thu chỉ bằng external signal:

- Goal quá mơ hồ, chẳng hạn “làm product experience tốt hơn” hoặc “nghĩ một growth strategy”.
- Verification signal yếu, chỉ có thể để Agent tự nói “tôi nghĩ đã ổn”.
- Nếu làm sai sẽ ảnh hưởng lớn, chẳng hạn write operation vào production database, thay đổi permission system hoặc cải tạo payment flow.
- Phụ thuộc mạnh vào thẩm mỹ và business judgment của con người, chẳng hạn định hướng brand copy hoặc trade-off của product phức tạp.
- Dự án cũ cải tạo lớn nhưng không có test, log hay rollback method.

“Experience tốt hơn” hoặc “tiếp tục tối ưu” không thể trigger stop judgment đáng tin cậy. Trước khi lặp, hãy tách goal thành các subtask có thể kiểm tra và viết ra tiêu chí phán định.

## Những điểm dễ mắc lỗi nhất

### Viết goal quá mơ hồ

“Tiếp tục tối ưu một chút” không có modification scope, verification command và stop condition, nên Agent không thể dựa vào đó để kết thúc task.

Cách viết chỉ có goal:

```text
/goal "Tối ưu project này để chất lượng code tốt hơn"
```

Cách viết có execution scope và exit condition:

```text
/goal "Tất cả unit test failure của module auth đều pass, chỉ được sửa src/auth và tests/auth; sau mỗi vòng sửa phải chạy npm test -- tests/auth và hiển thị exit code; tối đa 5 vòng; nếu nguyên nhân failure giống nhau trong 2 lần liên tiếp thì dừng và báo cáo"
```

`src/auth`, `tests/auth`, test exit code, giới hạn 5 vòng và điều kiện failure liên tiếp đều có thể được program kiểm tra; command thứ hai không để acceptance criteria cho Agent tự đoán.

### Lấy Agent tự đánh giá làm nghiệm thu

Test exit code, CI status, lint, type check hoặc screenshot comparison mới là completion evidence; phần giải thích của Agent chỉ có thể làm bổ sung. Khi cần semantic review, có thể để một Agent implementation và Agent khác kiểm tra trong context độc lập; với bước gần production thì thêm human approval.

### Quên giới hạn cost

Mỗi vòng đều có thể phải đọc lại file, gọi tool, giải thích error, nén context hoặc khởi động review Agent. Task config cần đồng thời giới hạn budget và stop condition:

- Số vòng iteration tối đa.
- Số lần tool call tối đa.
- Token / money budget theo ngày hoặc theo task.
- Phát hiện không có progress, chẳng hạn dừng khi nguyên nhân failure giống nhau trong hai vòng.
- Task giá trị thấp chỉ summary, không tự động repair.

No-progress detection dùng để ngăn việc tiếp tục tiêu tốn Token khi nguyên nhân failure không đổi.

### Cấp permission quá lớn

Đọc log, tạo PR và tự động merge PR có mức rủi ro hoàn toàn khác nhau, không thể dùng cùng một bộ permission để allow. Các operation như xóa file, sửa production config, gửi external message và ghi database mặc định đều phải chờ con người xác nhận.

Source của MCP Server, tool description, nội dung trả về và Prompt template cũng phải nằm trong phạm vi review, vì tất cả đều có thể mang prompt injection. Permission có thể mở dần theo từng execution stage:

| Stage                | Agent có thể làm gì                      | Con người phụ trách gì            |
| -------------------- | ---------------------------------------- | --------------------------------- |
| L0 Read-only summary | Đọc log, đọc Issue, tạo report           | Quyết định có chấp nhận hay không |
| L1 Local reproduce   | Chạy test được chỉ định, định vị failure | Quyết định có sửa hay không       |
| L2 Draft repair      | Sửa code trong worktree, chạy test       | Review diff                       |
| L3 Tạo PR            | Commit branch, viết PR description       | Review, merge                     |
| L4 Auto merge        | Tự động merge sau khi pass policy        | Chỉ xử lý exception               |

L1/L2 bao phủ việc đọc log, reproduce issue và sửa draft. L4 cần đồng thời thỏa mãn loại issue cố định, test bao phủ risk chính và rollback path đã được verify; task liên quan đến business judgment, permission hoặc data write vẫn phải giữ human approval.

![Ranh giới an toàn của Loop](https://oss.javaguide.cn/github/javaguide/ai/agent/loop-engineering-safety-boundary.webp)

## Version đầu tiên, đừng vội tự động repair

Khi làm Loop lần đầu, không nên trực tiếp bật unattended automatic repair. Có thể bắt đầu từ read-only triage, task description thà viết chi tiết hơn:

```text
Task: Mỗi ngày kiểm tra CI failure trong 24 giờ gần nhất và tạo summary troubleshooting để con người xử lý.

Được phép làm:
- Đọc GitHub Actions, commit gần nhất, log của test failure và file liên quan trực tiếp đến error.
- Khi định vị được test cụ thể, có thể chạy test tương ứng để xác nhận có reproduce hay không.
- Ghi conclusion vào TODO.md, kèm CI link, error chính và owner được đề xuất.

Trước khi bắt đầu:
- Trước hết đọc AGENTS.md.
- Khi match task CI troubleshooting thì load Skill ci-triage.

Không được làm:
- Không sửa code, không tạo PR, không gửi Slack/email.
- Không đọc file không liên quan trong toàn bộ repository, không paste full log.
- Dừng khi có hơn 10 failure item; mỗi failure chỉ được reproduce tối đa 2 lần.
- Khi thiếu permission, thiếu log hoặc cần business judgment thì trực tiếp đánh dấu xử lý thủ công.
```

Version này siết chặt action: chỉ xem CI evidence và file liên quan, không sửa code cũng không gửi external message; số failure item và số lần reproduce cũng có giới hạn. Trong troubleshooting summary, trước hết cần tập trung vào ba loại issue: file không liên quan, tool call lặp và action vượt permission. Khi chưa xử lý sạch ba loại này thì không nên mở repair permission.

Khi task kéo dài qua ngày, session mới sẽ không tự biết hôm qua đã thử gì. Nội dung cần khôi phục phải được ghi trực tiếp vào external state: goal, scope, evidence, action đã thực hiện, result, bước tiếp theo và stop condition đều phải để vòng sau đọc được.

```yaml
loop_id: ci-triage-2026-06-17
goal: "Điều tra CI failure trong 24 giờ gần nhất"
status: running
scope:
  repos: ["backend-service"]
  max_items: 10
attempts:
  - item: "auth-test failure"
    evidence: "GitHub Actions run #12345"
    action: "ran npm test -- tests/auth"
    result: "reproduced locally"
next_step: "ask auth owner to review"
stop_condition:
  max_attempts: 3
  require_human_when:
    ["permission_missing", "production_change", "uncertain_root_cause"]
```

Sau khi triage flow này chạy ổn, hãy thêm bốn hard limit cho automatic repair:

- Tạo worktree hoặc branch độc lập trước khi sửa.
- Whitelist modification scope.
- Chỉ được chạy test và formatting command đã chỉ định.
- Sau khi pass chỉ mở draft PR, không tự động merge.

## Trước khi đưa Loop vào production flow

### Điều gì sẽ trigger vòng tiếp theo?

Scheduled task hằng ngày sẽ tìm failure trong khoảng thời gian gần đây; CI failure event có thể trực tiếp giao failure job, PR liên quan và commit gần nhất cho triage. Task scope, file có thể đọc và command có thể dùng nên được truyền cùng task lần này, để session mới dựa vào đó chuẩn bị context.

### Khi nào tiếp tục, khi nào dừng?

CI link, failure log, command reproduce tối thiểu và test exit code có thể cho biết troubleshooting có progress mới hay không; `attempts` và `stop_condition` giữ action đã thử cùng nguyên nhân dừng trong external state. Khi thiếu permission, không thể reproduce, cần business judgment hoặc failure liên tiếp đạt giới hạn, task nên quay lại human queue. Tạo PR, sửa production config và auto merge không nên là subsequent action mặc định của cùng một Loop.

## Tổng kết

Loop Engineering nối mỗi call với evidence, state và giới hạn mà vòng trước để lại.

CI triage trong bài viết bắt đầu từ việc đọc log, định vị failure và ghi conclusion; YAML state lưu `evidence`, `action`, `result` và `next_step`. Các field này giúp vòng sau tiếp tục xử lý ngay, đồng thời giúp con người biết đã làm gì khi takeover. Sau khi read-only troubleshooting ổn định, mới từng bước mở permission sửa code, tạo PR và auto merge.
