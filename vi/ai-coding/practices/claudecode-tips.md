---
title: "Hướng dẫn sử dụng Claude Code: cấu hình, workflow và kỹ thuật nâng cao"
description: "Kết hợp tài liệu chính thức của Anthropic và cách dùng trong dự án thực tế để giải thích rõ cấu hình, quyền hạn, MCP, Skills, Sub-Agent, quản lý context và các workflow thường gặp của Claude Code."
category: AI coding thực chiến
head:
  - - meta
    - name: keywords
      content: Claude Code,AI coding,CLAUDE.md,MCP,Skills,Sub-Agent,Agentic Coding,AI-assisted development
---

Xin chào, tôi là Tiểu G. Vài ngày trước, bài [Tổng hợp các kỹ thuật Vibe Coding thực dụng](./the-cool-tricks-for-vibe-coding.md) đạt hơn 60 nghìn lượt đọc trong hai ngày trên tài khoản công khai, và khá nhiều người hỏi về Claude Code trong phần bình luận.

Bài này sẽ nói riêng về Claude Code.

Không biết mọi người có cảm giác giống tôi không: lúc mới dùng thật sự khá gượng gạo, thậm chí hơi chống đối. Đã quen với sidebar, file tree và diff panel trong Cursor, IDEA, rồi quay lại phối hợp với AI trong terminal thì thực sự không thuận tay.

Sau khi dùng nhiều hơn, tôi lại thấy lớp CLI rất phù hợp với các task dài. Nó có thể chạy local, cũng có thể chuyển lên máy remote, môi trường tạm thời hoặc CI/CD; cùng một bộ command, quyền hạn và cách verify có thể tái sử dụng, không cần sửa lại các bước chỉ vì GUI.

Hiện tại khi dùng Claude Code, trước tiên tôi nói rõ directory, mục tiêu và cách nghiệm thu, để nó tự đọc code, chạy command, cuối cùng tôi xem diff và kết quả test.

Rắc rối cũng nằm ở đây. Viết `CLAUDE.md` quá đầy đủ, cấp quyền quá rộng, nhồi quá nhiều context, chia sai ranh giới Sub-Agent đều có thể khiến nó càng chạy càng lệch hướng.

Nội dung dưới đây phần lớn đến từ ghi chép sử dụng của tôi trong hơn một năm, thiên về thực chiến, không cố kể lại tài liệu chính thức từ đầu.

PS: Claude Code thay đổi rất nhanh. Bài viết này được tổng hợp theo tài liệu chính thức và kinh nghiệm cá nhân với v2.1.218 (2026-07-24). Hành vi của command, permission mode, plugin, Auto Mode, Sub-Agent và Worktree có thể chịu ảnh hưởng bởi version, platform, gói tài khoản, provider và kênh cài đặt. Trước khi dùng thực tế, tốt nhất hãy xem `claude --version`, `claude --help`, `/help` và tài liệu chính thức. Ví dụ, `/run`, `/verify` cần v2.1.145+; `/code-review` hỗ trợ mức effort, `--comment` và `--fix`; `/simplify` hiện phù hợp hơn khi hiểu là cleanup-only review, không phải correctness bug review đầy đủ.

Khi sử dụng trong nước, cũng cần cân nhắc tài khoản, network, chi phí và độ ổn định của bên trung chuyển thứ ba. Các model nội địa như GLM, MiniMax, Kimi, DeepSeek có thể dùng làm lựa chọn thay thế hoặc bổ sung; nhưng khi gặp thay đổi code quy mô lớn, refactor phức tạp hay debug theo chuỗi dài, Claude hiện vẫn đáng để nghiên cứu riêng.

## `CLAUDE.md` rất quan trọng

Tốt nhất đừng viết `CLAUDE.md` thành một README thứ hai. Nó giống một bản ghi nhớ dự án viết cho Claude Code hơn: những rule không thể nhìn ra từ code, những command nó thường đoán sai, những directory không được chạm vào, và loại test nào bắt buộc phải chạy sau khi sửa một loại code nào đó.

![CLAUDE.md và AGENTS.md trong dự án phân tích cổ phiếu bằng AI](https://oss.javaguide.cn/github/javaguide/ai/coding/claude-agents-md.png)

Trong các file dự án, tôi thường chỉ giữ những thứ sau: **rule Claude dễ đoán sai, quy ước không thể đọc ra từ code, quy chuẩn team bắt buộc tuân thủ, cùng với version tech stack, command thường dùng, các đánh đổi kiến trúc và điểm dễ mắc lỗi của dự án.**

Tài liệu chính thức khuyến nghị kiểm soát mỗi `CLAUDE.md` trong phạm vi 200 dòng. File quá dài sẽ tiêu tốn nhiều context hơn, cũng có thể làm giảm mức độ tuân thủ rule. Khi nội dung tiếp tục phình ra, hãy tách sang `.claude/rules/` có `paths`, còn nội dung tham khảo ít dùng thì đưa vào Skills.

![Khuyến nghị của tài liệu chính thức Claude Code về CLAUDE.md](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/claudemd-claude-docs.png)

Để phán đoán một rule có nên giữ lại hay không, tôi thường tự hỏi:

> Nếu xóa dòng này, Claude có dễ mắc lỗi hơn không?

Nếu có thì giữ lại; nếu không thì xóa ngay.

### Đặt ở đâu

`CLAUDE.md` có thể được đặt ở nhiều vị trí. Thứ tự load chính thức nhìn chung đi từ global đến local, đừng chỉ nhìn file ở root project:

![Cấp bậc và mức ưu tiên của CLAUDE.md](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/claude-md-best-practices-file-hierarchy.png)

Ngoài cùng là file cấp tổ chức, thường do IT hoặc DevOps phân phối thống nhất. Path trên macOS là `/Library/Application Support/ClaudeCode/CLAUDE.md`, trên Linux/WSL là `/etc/claude-code/CLAUDE.md`, trên Windows là `C:\Program Files\ClaudeCode\CLAUDE.md`. Các rule kiểu này thường không phải thứ cần sửa trong project cá nhân.

Tiếp theo là `~/.claude/CLAUDE.md` cấp user, phù hợp để đặt preference chung của bản thân. File cấp project đặt tại `./CLAUDE.md` hoặc `./.claude/CLAUDE.md`, nên được commit vào Git để cả team đều thấy. `./CLAUDE.local.md` cấp local chỉ giữ cấu hình cá nhân, nhớ thêm nó vào `.gitignore`. `CLAUDE.md` trong subdirectory sẽ không được nhồi toàn bộ vào context ngay từ đầu; chỉ được load khi Claude truy cập đến directory tương ứng.

Các file này cùng đi vào context, file được load sau không ghi đè toàn bộ nội dung phía trước. Chỉ là rule càng gần project hiện tại và phạm vi tác động càng cụ thể thì càng nằm về sau, Claude thường cũng dễ tiếp nhận hơn.

Ví dụ rule cấp user viết “dùng indentation bằng space thống nhất”, rule cấp project viết “repository này dùng Tab”, thì trong project này Claude thường ưu tiên theo rule của project. Thứ tự load trong tài liệu chính thức cũng đi từ cấp tổ chức, cấp user, rồi đến cấp project và cấp local.

Thói quen của tôi là commit `CLAUDE.md` cấp project vào Git và chỉ viết các rule team cùng tuân thủ. Preference chỉ liên quan đến bản thân, ví dụ muốn message commit bằng tiếng Việt trong một project nào đó, thì đưa vào `CLAUDE.local.md`, rồi thêm vào `.gitignore`.

Khi project có quy mô lớn, có thể tách ra:

```text
my-project/
├── CLAUDE.md
├── backend/
│   └── CLAUDE.md
├── frontend/
│   └── CLAUDE.md
└── .claude/
    ├── rules/
    ├── skills/
    └── agents/
```

Root directory đặt quy ước chung, subdirectory đặt rule cục bộ. Khi đọc đến file của một subdirectory, Claude sẽ load mô tả trong directory tương ứng khi cần. Cơ chế này rất phù hợp với monorepo: backend, frontend và trang quản trị không cần chen chúc trong một file.

Đừng hiểu nhầm tham chiếu `@path`. Nó không tự nhiên làm giảm context; nội dung được tham chiếu cuối cùng vẫn được đưa vào, chỉ là dễ bảo trì hơn. Khi một số rule chỉ có hiệu lực với directory cụ thể, hãy ưu tiên các rule load theo path như `.claude/rules/`, đừng tiếp tục nhồi chúng vào file root.

### Khởi tạo và bảo trì

Project mới có thể chạy trước:

```bash
/init
```

Claude sẽ đọc repository và tạo một `CLAUDE.md` ban đầu. Chỉ nên xem file này là bản nháp, đừng commit ngay. Nó có thể đoán sai command build, cũng có thể chép lại một lần nữa nội dung đã được viết rõ trong README.

Khi bảo trì, điều dễ mất kiểm soát nhất là viết ngày càng nhiều.

Claude thỉnh thoảng mới mắc một lỗi thì đừng vội thêm rule. Hãy đợi vấn đề cùng loại xuất hiện hai, ba lần, và bạn có thể chặn nó bằng một instruction rõ ràng, rồi mới viết vào. Ngược lại, sự thật có thể đọc ra ngay từ code, việc model vốn sẽ tự làm, hay quy ước lịch sử đã lỗi thời đều nên xóa. Khi có quá nhiều rule, vài câu quan trọng nhất sẽ bị loãng.

Nếu Claude rõ ràng đã đọc rule nhưng không làm theo, trước hết hãy xem cách viết rule có quá mềm không. “Cố gắng giữ test đầy đủ” rất mơ hồ; “Sau khi sửa Service bắt buộc chạy unit test tương ứng, đồng thời ghi rõ command và kết quả” dễ thực thi hơn. Nếu cùng một rule liên tục thất bại trong nhiều session, hãy kiểm tra xem file có quá dài không hoặc rule có đặt sai vị trí không.

Tôi chia rule thành hai loại: yêu cầu cấp team, có hiệu lực dài hạn và bắt buộc chia sẻ thì viết vào `CLAUDE.md`; preference cá nhân, kinh nghiệm debug theo giai đoạn và lời nhắc tạm thời thì giao cho Auto Memory hoặc cấu hình local. `CLAUDE.md` tốt nhất nên xuất phát từ lỗi thực tế, đồng thời định kỳ xóa nội dung đã mất hiệu lực.

Sau khi viết rule, cũng đừng mặc định rằng nó đã có hiệu lực. Dùng `/context` để xem session hiện tại thực sự đã load những `CLAUDE.md`, `CLAUDE.local.md` và file rules nào; `/memory` chủ yếu dùng để xem và chỉnh sửa vị trí cấu hình của rule và memory. Nếu một file không xuất hiện trong kết quả `/context`, Claude sẽ không thấy nó trong lượt này. Trong project phức tạp có dùng `.claude/rules/` kèm `paths`, còn có thể dùng `InstructionsLoaded` Hook để ghi lại thời điểm và lý do file rule được load.

## Cần coi trọng quản lý quyền hạn

### Cấp quyền theo tầng

Theo mặc định, Claude Code sẽ yêu cầu xác nhận với các thao tác nhạy cảm, chẳng hạn ghi file, thực thi Bash và gọi MCP tool. Lúc đầu có thể thấy phiền, nhưng khi chưa quen với cách thực thi của nó, giữ bước xác nhận trước sẽ an toàn hơn.

Thông thường tôi chỉ mở những command xem cũng không gây sự cố, chạy cũng không phá hiện trường. Ví dụ command chỉ đọc như `git diff`, `git status`, `rg` có thể bớt chặn; các command verify cố định như `mvn test`, `pnpm test`, `npm run lint` cũng có thể cho phép tùy tình hình project.

Ngược lại, mặc định không nên cho phép các thao tác như `rm -rf`, `git push --force`, sửa `.git/`. Với `.env`, `secrets/`, artifact sinh ra, directory chứa certificate và các file dump, cũng nên dùng rule deny để chặn trước.

Có thể cấu hình quyền qua `/permissions`, hoặc ghi vào `.claude/settings.json`:

```json
{
  "permissions": {
    "allow": ["Bash(git status*)", "Bash(git diff*)", "Bash(rg *)"],
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Bash(rm -rf *)"
    ]
  }
}
```

Các rule sẽ được tầng thực thi của Claude Code xử lý. Nghĩa là dù trong prompt có viết “nhất định không được đọc `.env`”, đó vẫn chỉ là lời khuyên; chỉ rule deny mới chặn được thao tác tương ứng.

Classifier của Auto Mode cũng tham khảo ranh giới bạn viết trong hội thoại, nhưng đây không phải bảo đảm cứng. Ranh giới không được phép bỏ qua tốt nhất nên viết vào `permissions.deny`, hoặc dùng Hook để chặn trước khi gọi tool. Sau khi session dài được compress, các giới hạn tạm thời từng nói trong chat cũng có thể bị loại bỏ.

### Auto Mode

Nếu việc xác nhận thường xuyên đã ảnh hưởng nhịp độ, có thể cân nhắc Auto Mode.

Trong tài liệu chính thức hiện tại, CLI chuyển permission mode bằng `Shift+Tab`; chỉ khi account, model, provider và cấu hình tổ chức đều đáp ứng yêu cầu thì `auto` mới xuất hiện trong vòng lặp mode. Trong môi trường Team / Enterprise, administrator còn có thể bật hoặc khóa nó.

Nguyên lý của nó là dùng một classifier riêng để phán đoán rủi ro thao tác, tự động cho phép thao tác rủi ro thấp; các hành động như tải xuống và thực thi code lạ, gửi nội dung nhạy cảm đến endpoint bên ngoài, deploy production, force push, push trực tiếp vào `main` sẽ bị chặn hoặc chuyển sang xác nhận thủ công.

Tuy nhiên Auto Mode không cung cấp sandbox an toàn và không bảo đảm không phán đoán sai. Nó giải quyết việc “bớt xác nhận”, không chịu trách nhiệm cô lập file system, network và credential. Task rủi ro cao vẫn phải dựa vào container, account tạm thời, quyền tối thiểu, rule deny, Hooks và Review thủ công.

Nếu muốn mặc định vào Auto Mode, cũng đừng ghi `"defaultMode": "auto"` vào `.claude/settings.json` hoặc `.claude/settings.local.json` cấp project. Từ v2.1.142+, các thiết lập `auto` trong những nguồn này sẽ bị bỏ qua, tránh việc repository tự mở Auto Mode cho chính nó. Nên đặt nó trong `~/.claude/settings.json` cấp user hoặc managed settings cấp tổ chức. Các provider như Bedrock, Vertex AI, Microsoft Foundry còn có thể cần thiết lập bổ sung `CLAUDE_CODE_ENABLE_AUTO_MODE=1`.

Cũng không nên hard-code argument khởi động. Hỗ trợ permission mode có thể khác nhau giữa version, kênh cài đặt và provider; trong script tốt nhất hãy dùng `claude --help` hoặc tài liệu chính thức để xác nhận value hiện có. Khi dùng interactive, tôi thiên về việc dùng `Shift+Tab` để chuyển mode trong session thay vì ghi mode quyền cao vào script.

Tôi không khuyến nghị dùng `--dangerously-skip-permissions` trong project thường ngày. Trừ khi đã cô lập file system, network và credential, nếu không chỉ một thao tác nhầm cũng có thể sửa nhầm file hoặc đọc nhầm credential.

## Ranh giới bảo mật

Không được trực tiếp để Claude tiếp xúc với credential production, password database hay token dài hạn của cloud provider; cũng đừng để nó trực tiếp chạm vào môi trường production, trừ khi việc đó vốn đã có approval và audit.

Phía Git cũng cần siết chặt. Đừng mặc định cho phép nó push vào `main`, càng không nên biến force push branch remote thành thao tác có thể thực hiện tùy tiện. Với script remote không rõ nguồn gốc, đặc biệt là cách viết `curl | bash`, tốt nhất chỉ thử trong môi trường cô lập.

Phạm vi đọc file cũng phải được kiểm soát. `.env`, `secrets/`, directory chứa certificate, SSH key, database dump và log production đều không nên mặc định nằm trong phạm vi Claude có thể đọc.

Không chỉ có `.env`. Những nơi như `~/.aws/`, `~/.gcp/`, `~/.kube/`, `~/.ssh/`, `settings.xml` của Maven, npm token, log production và database dump cũng không nên tùy tiện để Claude tiếp xúc. Nếu thực sự cần nó xem log, tốt nhất hãy khử thông tin nhạy cảm, cắt lấy phần cần thiết và giới hạn phạm vi trước.

Khi thực sự cần tự động hóa task quyền cao, hãy chạy trong container, với credential tạm thời và account có quyền tối thiểu. Như vậy ngay cả khi thực thi sai command, phạm vi ảnh hưởng cũng dễ kiểm soát hơn.

## Phân biệt MCP, Skills, Sub-Agent và plugin

Các thành phần xung quanh Claude Code khá nhiều, lúc mới tiếp xúc rất dễ trộn lẫn. Cách phân loại của tôi đại khái như sau:

| **Cơ chế**  | **Giải quyết vấn đề gì**                 | **Phù hợp để đặt gì**                                              | **Không phù hợp để đặt gì**             |
| ----------- | ---------------------------------------- | ------------------------------------------------------------------ | --------------------------------------- |
| `CLAUDE.md` | Bối cảnh cần biết trong mỗi session      | Command build, quy ước directory, rule team                        | Quy trình task nhiều bước               |
| Rules       | Rule cục bộ load theo path               | Rule frontend, rule backend, rule bảo mật                          | Quy ước cốt lõi cả project đều phải xem |
| Skills      | Các bước task có thể tái sử dụng         | TDD, Code Review, viết bài, triển khai frontend                    | Kiến thức nền tảng lâu dài              |
| MCP         | Kết nối hệ thống bên ngoài               | GitHub, Sentry, Notion, Figma, database                            | Rule về file local thông thường         |
| Sub-Agent   | Cô lập context của task nhánh            | Tìm kiếm code, review chuyên biệt, nghiên cứu song song            | Thay đổi một lần có ranh giới rất nhỏ   |
| Hooks       | Hành động thực thi cố định               | Cấm command nguy hiểm, format sau khi sửa, test trước khi kết thúc | Lời khuyên chỉ mang tính tham khảo      |
| Plugin      | Đóng gói và phân phối một nhóm extension | Tổ hợp Skills, MCP, Hooks, script                                  | Gói quyền hạn bên thứ ba chưa review    |

Ví dụ “sau mỗi lần edit bắt buộc chạy formatter”, viết vào `CLAUDE.md` chỉ có thể nhắc Claude nhớ, còn viết thành Hook mới có thể trigger sau khi file được sửa. Tương tự, “quy trình xử lý GitHub Issue” đặt trong `CLAUDE.md` sẽ làm nhiễu mọi session, phù hợp hơn khi làm thành Skill.

### Code Intelligence: giúp Claude bớt phụ thuộc vào việc đọc bằng full-text search

Nếu project có thể dùng Code Intelligence thì nên cấu hình. Nó tương đương việc gắn cho Claude một language server: xem lỗi type, tìm định nghĩa symbol, tra quan hệ tham chiếu, không cần lần nào cũng dùng `rg` tìm trong một mảng file lớn.

Ví dụ với project Java hoặc TypeScript, khi Claude muốn biết một class được định nghĩa ở đâu, được ai gọi, không nhất thiết phải tìm keyword trước rồi lần lượt mở file để xác nhận. Nhờ LSP, nó có thể nhảy thẳng đến definition, xem reference, sau khi sửa code còn có thể lập tức phát hiện lỗi type.

Nó không thể thay thế full-text search, nhưng có thể giảm lượng file không liên quan phải đọc. Khi project lớn, context sẽ sạch hơn đáng kể, Claude cũng ít bị làm lệch hướng bởi một đống file ứng viên. Tài liệu chính thức của Claude Code cũng khuyến nghị ngôn ngữ có type nên cài plugin Code Intelligence, vì một lần nhảy symbol thường tiết kiệm được một lần search và nhiều lần đọc file.

Tuy nhiên plugin Code Intelligence không phải “cài xong là xong”. Máy local còn phải có binary language server tương ứng, chẳng hạn Java cần `jdtls`, TypeScript cần `typescript-language-server`. Nếu trang Errors của `/plugin` xuất hiện `Executable not found in $PATH`, thường là dependency này chưa được cài đúng.

### MCP: giúp Claude kết nối với thế giới thực

MCP (Model Context Protocol, giao thức context của model) phụ trách kết nối hệ thống bên ngoài. Hệ thống bên ngoài cung cấp một MCP Server; sau khi client như Claude Code kết nối, nó có thể nhìn thấy và gọi các tool bên trong.

![Sơ đồ MCP](https://oss.javaguide.cn/github/javaguide/ai/skills/mcp-simple-diagram.png)

Đây là cách chính để Claude Code kết nối tool bên ngoài. Tra database, đọc lỗi Sentry, truy cập browser, lấy tài liệu Notion hay lấy design từ Figma đều thuộc nhóm này.

Command thêm remote MCP server đại khái như sau:

```bash
claude mcp add --transport http notion https://mcp.notion.com/mcp \
  --header "Authorization: Bearer your-token"
```

`your-token` ở đây chỉ mang tính minh họa. Trong project thực tế, cố gắng đừng ghi token trực tiếp vào shell history.

Trong project team, cấu hình MCP có thể chia sẻ nên đặt trong `.mcp.json`, sau đó commit vào repository. Ví dụ một project thống nhất cần kết nối Notion, Sentry và hệ thống tài liệu nội bộ thì có thể lưu lại tên server, URL, transport và các cấu hình chung khác.

Cấu hình chứa token, secret hoặc connection string database không được commit vào `.mcp.json`. Cách ổn định hơn là đặt trong cấu hình cấp user, biến môi trường local, hệ thống quản lý secret, hoặc dùng OAuth flow được MCP server tương ứng hỗ trợ.

Hãy tiết chế MCP Server. Càng nhiều tool, Claude càng dễ chọn sai và càng khó audit. Có thể dùng `/mcp` để xem trạng thái kết nối hiện tại, enable hoặc disable server; để xem chi phí và usage theo từng phần thì phù hợp hơn với `/usage`, nó sẽ hiển thị usage theo các chiều như skill, subagent, plugin và MCP server. Server không thường dùng thì nên disconnect trước.

### Skills: lưu lại các thao tác lặp lại

Đừng trộn rule file với Skill.

Rule file dùng cho constraint dài hạn, chẳng hạn version tech stack, command khởi động, cấu trúc directory, format error code và những file không được chạm vào.

Skill dùng cho các bước task, chẳng hạn code review, viết test, sửa frontend, nghiên cứu web và viết bài kỹ thuật. Cách thực hiện các task này mỗi lần gần như giống nhau, không cần lặp lại lời nhắc trong chat.

Trước đây Tiểu G đã viết hai bài liên quan: [Agent Skills là gì? Khác Prompt và MCP ở đâu?](https://javaguide.cn/ai/agent/skills.html) và [Danh sách lựa chọn Skills cho AI coding](https://javaguide.cn/ai-coding/practices/programmer-essential-skills.html).

Skill là một tài liệu task được load khi cần. Cách thực hiện một loại task, các constraint, những điểm cần kiểm tra và các lỗi từng gặp đều được viết vào `SKILL.md`.

Một điểm khác giữa nó và `CLAUDE.md` là thời điểm load. Theo mặc định Claude chỉ thấy tên và mô tả của Skill để phán đoán có nên gọi hay không; khi gọi Skill đó, phần nội dung `SKILL.md` và resource liên quan mới được đưa vào context. Skill cấp user đặt tại `~/.claude/skills/`, Skill cấp project đặt tại `.claude/skills/`.

Cũng cần chú ý một thay đổi về version: trong Claude Code, custom commands đã được gộp vào Skills. `.claude/commands/deploy.md` và `.claude/skills/deploy/SKILL.md` đều có thể tạo command dạng `/deploy`; `.claude/commands/` cũ vẫn dùng được, nhưng nội dung mới được khuyến nghị tổ chức theo Skill.

![Chuỗi thực thi Agent](https://oss.javaguide.cn/github/javaguide/ai/skills/skills-agent-execution-link.png)

Các bước có tính lặp lại cao đều có thể tích lũy thành Skill. Trước khi viết feature thì cố định đi theo TDD, viết test fail trước rồi mới implement; khi code review thì cố định kiểm tra bảo mật, transaction, performance và điều kiện biên; khi viết bài kỹ thuật thì cố định đối chiếu nguồn sự thật, citation, cấp heading và dấu vết AI.

Cách này ổn định hơn nhiều so với việc bổ sung một chuỗi nhắc dài trong prompt mỗi lần. Định nghĩa chính thức về Skill cũng gần với ý này: một nhóm instruction, script và resource có thể tái sử dụng, giúp Claude xử lý một loại task theo các bước cố định.

Cũng có thể dùng Skill có sẵn, chẳng hạn Superpowers đã đóng gói các bước TDD, Code Review, Spec-Driven, Git Worktree và phối hợp giữa các sub Agent.

Tôi có đề xuất chi tiết trong bài [Danh sách lựa chọn Skills cho AI coding: làm rõ yêu cầu, TDD, code review và UI design](https://javaguide.cn/ai-coding/practices/programmer-essential-skills.html).

Đừng lấy Skill bên thứ ba về rồi chạy ngay. Bản thân `SKILL.md` đã là instruction; nếu bên trong có command nguy hiểm, script kỳ lạ hay quyền quá rộng, Agent có thể làm theo. Trước khi cài, ít nhất hãy xem phần nội dung, `scripts/` và `references/`, xác nhận nó không thực hiện thao tác vượt quyền.

### Plugin: xem marketplace chính thức trước

Nếu không muốn tự cấu hình Skills, MCP và Hooks từ đầu, có thể vào marketplace plugin chính thức của Claude Code là [`claude-plugins-official`](https://github.com/anthropics/claude-plugins-official) để xem trước.

Cài đặt cũng rất trực tiếp:

```bash
/plugin install <name>@claude-plugins-official
```

Plugin giúp tiết kiệm thời gian lắp ghép. Một plugin có thể đã đóng gói sẵn Skill, MCP Server, Hooks và một số script hỗ trợ; sau khi cài, Claude có thêm một workflow hoàn chỉnh có thể dùng ngay.

Nhưng cuối cùng plugin vẫn chạy trên máy local của bạn, một số plugin còn chạm vào file system, browser, GitHub, database hoặc service bên thứ ba. Trước khi cài, ít nhất hãy xem mô tả, quyền hạn và nguồn source; không dùng nữa thì gỡ để giảm các entry point tool không cần thiết. Cách cài đặt và tìm plugin cụ thể có thể xem trong tài liệu chính thức [Discover plugins](https://code.claude.com/docs/en/discover-plugins).

### Sub-Agent: giữ cho session chính sạch

Tôi dùng Sub-Agent khá thường xuyên.

![Claude Code Sub-Agent: giữ cho hội thoại chính sạch](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode-sub-agent.png)

Khi điều tra vấn đề phức tạp, Claude thường phải đọc hàng chục file, tìm một đống code và chạy vài command. Session chính nhanh chóng bị log, kết quả search và nội dung file lấp đầy; sau đó tiếp tục viết code rất dễ mất phương hướng.

Có thể giao các task nhánh như vậy cho Sub-Agent. Nó có context riêng, có thể tự đọc code, tra log và phân tích vấn đề; khi kết thúc chỉ báo cáo kết luận về session chính.

Trong các subagent built-in của Claude Code, những loại tôi thường gặp nhất là Explore, Plan và general-purpose. Các subagent built-in này đều kế thừa quyền của parent session, nhưng đồng thời áp dụng thêm giới hạn tool riêng:

| **Sub-agent**       | **Model**                              | **Tool/quyền hạn**                                                    | **Mục đích**                                       |
| ------------------- | -------------------------------------- | --------------------------------------------------------------------- | -------------------------------------------------- |
| **Explore**         | Haiku, thiên về tốc độ và latency thấp | Chỉ đọc, không có Write / Edit                                        | Phát hiện file, tìm kiếm code, khám phá codebase   |
| **Plan**            | Kế thừa model của hội thoại chính      | Chỉ đọc, không có Write / Edit                                        | Nghiên cứu codebase trong Plan Mode                |
| **general-purpose** | Kế thừa model của hội thoại chính      | Kế thừa tool session chính có thể dùng, vẫn chịu constraint quyền hạn | Nghiên cứu phức tạp, thao tác nhiều bước, sửa code |

Explore và Plan thiên về nghiên cứu chỉ đọc, không trực tiếp sửa code. Một chi tiết trong tài liệu chính thức: Explore và Plan bỏ qua file `CLAUDE.md` và git status của parent session, vì vậy phù hợp hơn để tìm code và thu thập context nhanh; các subagent built-in khác và subagent tự tạo sẽ load những nội dung này.

general-purpose có ranh giới rộng hơn, có thể khám phá, thực thi command và sửa code. Trước khi dùng, tốt nhất phải nói rõ directory nào được đọc, file nào không được sửa, có cho phép ghi hay không, cuối cùng chỉ cần kết luận hay phải trực tiếp triển khai. Nếu thực sự cần constraint chặt, không thể chỉ dựa vào prompt mà phải kết hợp `tools` / `disallowedTools` của subagent, permission mode, `permissions.deny` hoặc Hooks.

Bạn cũng có thể tạo subagent của riêng mình. Cấu hình cấp project đặt trong `.claude/agents/` để chia sẻ với team; cấu hình cấp user đặt trong `~/.claude/agents/` để tái sử dụng giữa các project. Mỗi subagent đều có thể cấu hình system prompt, quyền tool, model và điều kiện trigger.

Tình huống tôi thường dùng là để subagent chạy test suite, chỉ mang test case thất bại và thông tin lỗi về; hoặc để các subagent khác nhau lần lượt nghiên cứu module authentication, database và API, cuối cùng hợp nhất kết luận vào session chính. Với task phức tạp hơn, cũng có thể để subagent code-reviewer tìm vấn đề performance trước, sau đó để subagent optimizer thử sửa.

Task quá nhỏ, ranh giới không rõ hoặc code còn thay đổi mạnh thì không nhất thiết phải tách subagent. Session chính giữ mục tiêu, quyết định và tiêu chí nghiệm thu; subagent chỉ xử lý task chuyên biệt có phạm vi cục bộ, rõ ràng và có thể báo cáo kết quả.

Sau này khi dùng Agent teams, có thể xem đó là cách phối hợp nhiều session. Sub-Agent dùng để cô lập task nhánh; Agent teams dùng để nhiều session độc lập phối hợp xoay quanh task chung. Khi mới bắt đầu không cần vội, trước hết hãy làm quen với Worktree, commit nhỏ và nhịp verify.

Một subagent review bảo mật tự tạo có thể viết như sau:

```markdown
---
name: security-reviewer
description: Reviews Java and Spring Boot code for security risks.
tools: Read, Grep, Glob, Bash
model: opus
---

Review the target diff for:

- SQL injection and unsafe dynamic queries.
- Authentication and authorization bypass.
- Secrets or credentials committed to code.
- Unsafe deserialization or command execution.

Return concrete file and line references. Do not rewrite code unless explicitly asked.
```

Trong project thực tế, nên thu hẹp `tools` của subagent hết mức có thể. Nếu chỉ làm code review thì thường không cần `Edit` / `Write`. Khi đồng thời thiết lập `tools` và `disallowedTools`, `disallowedTools` sẽ được áp dụng trước; nếu cùng một tool xuất hiện ở cả hai bên thì cuối cùng sẽ bị loại bỏ.

### Hooks: xử lý các rule bắt buộc thực thi

Hooks rất dễ bị bỏ qua, nhưng trong project thực tế lại rất hữu ích. Nó có thể thực thi action tại các lifecycle node của Claude Code, chẳng hạn trước khi gọi tool, sau khi edit file, trước khi session kết thúc và trước hoặc sau khi context được compress.

Ví dụ, giả sử Claude Code chuẩn bị thực thi:

```bash
rm -rf /tmp/build
```

`PreToolUse` Hook sẽ nhận lần gọi Bash này trước, phán đoán xem nó có nguy hiểm không; sau khi match rule, nó trả về `deny`, Claude Code sẽ hủy lần gọi tool này và phản hồi lý do từ chối cho Claude.

Hình dưới đây lấy từ tài liệu Hooks chính thức của Claude Code, minh họa chính flow này.

![Claude Code PreToolUse Hook](https://oss.javaguide.cn/github/javaguide/ai/coding/claude-code-runs-rm-rf-tmp-build-what-happens.svg)

Tôi sẽ giao một số loại action cho Hook: tự động format sau khi edit, chạy test trước khi session kết thúc, cấm sửa `migrations/` hoặc `.github/workflows/`, chặn `curl | bash`, `rm -rf`, chặn gửi nội dung nhạy cảm đến endpoint bên ngoài, hoặc inject context bổ sung khi Sub-Agent khởi động.

Action cần thực thi cố định thì phù hợp để đặt vào Hook; nội dung chỉ dùng tham khảo làm background thì viết vào `CLAUDE.md`.

Nếu viết HTTP Hook, còn phải chú ý một điểm dễ mắc: không thể dựa vào việc trả về 4xx / 5xx để chặn tool call. Non-2xx, lỗi kết nối và timeout của HTTP Hook đều được xem là lỗi không blocking, execution vẫn tiếp tục. Để chặn tool call, cần trả về 2xx và ghi `decision: "block"` trong JSON, hoặc ghi `permissionDecision: "deny"` trong `hookSpecificOutput`.

## Workflow thường dùng nhất

### Khám phá, lập kế hoạch, thực thi, verify

Đừng vừa bắt đầu task phức tạp đã yêu cầu Claude viết code. Trước hết hãy để nó đọc repository và tạm thời không sửa file:

```text
Vào plan mode. Trước tiên đọc src/auth và các test liên quan để hiểu rõ flow refresh trạng thái đăng nhập.
Không viết code, chỉ báo cáo flow hiện tại, các file liên quan và những điểm có thể cần sửa.
```

Sau khi đọc xong mới yêu cầu nó đưa ra plan:

```text
Tôi muốn sửa lỗi refresh token thất bại sau khi session của user timeout.
Dựa trên phần đã đọc, hãy liệt kê các file cần sửa, strategy test và các điểm rủi ro.
```

Sau khi bạn xác nhận plan mới thực thi:

```text
Triển khai theo plan này. Ưu tiên bổ sung một test có thể reproduce vấn đề trước, rồi mới sửa implementation.
Sau khi hoàn tất hãy chạy test liên quan, ghi rõ command và kết quả.
```

Nhịp này ban đầu sẽ chậm hơn một chút, nhưng sau đó giảm rất nhiều việc làm lại. Đặc biệt khi chưa quen codebase hoặc thay đổi trải rộng nhiều module, trước hết để Claude nói rõ call chain, các điểm rủi ro và strategy test sẽ tránh được nhiều tình huống “sửa xong mới phát hiện đi sai hướng”.

Thay đổi nhỏ có thể bỏ qua plan. Ví dụ sửa một đoạn text, thêm một log hoặc bổ sung một kiểm tra null pointer nhìn là thấy ngay thì cứ yêu cầu nó làm trực tiếp. Lập plan quá mức cũng lãng phí context.

### TDD, test-driven development

Điểm phiền nhất khi AI viết code là nó rất giỏi viết code “trông có vẻ hợp lý”. TDD có thể chốt hành vi mong đợi trước, rồi để implementation tiến gần đến test.

Prompt không cần vòng vo:

```text
Trước tiên đừng sửa implementation. Hãy viết một test thất bại cho TokenRefreshService,
bao phủ trường hợp session đã hết hạn nhưng refresh token vẫn còn hiệu lực.
Sau khi test thất bại thì sửa implementation cho đến khi test pass.
```

Nếu test chưa từng thất bại, rất khó xác nhận implementation phía sau thực sự đã sửa đúng vấn đề nào. Nếu không, nó có thể sửa một đống code rồi nói với bạn “đã sửa xong”.

Superpowers được đề xuất trong [Danh sách lựa chọn Skills cho AI coding](https://javaguide.cn/ai-coding/practices/programmer-essential-skills.html) cũng đã đóng gói TDD.

### Để Claude tự verify

Trong best practice chính thức của Anthropic có một câu tôi rất đồng tình: **Hãy đưa cho Claude một bước kiểm tra có thể chạy được. Test, build, lint, so sánh screenshot hay output của script đều được.**

Ví dụ đừng chỉ nói:

```text
Viết một function validate email.
```

Viết như dưới đây sẽ giảm rất nhiều phỏng đoán:

```text
Viết một function validate email. Test case:

- hello@gmail.com phải pass
- hello@ phải fail
- @domain.com phải fail
- a@b.co phải pass

Sau khi viết xong hãy chạy test, ghi rõ command và kết quả.
```

Tiêu chí nghiệm thu càng cụ thể, Claude càng ít dừng ở trạng thái “trông có vẻ đã hoàn thành”.

Nếu task sẽ chạy lâu, có thể thêm câu “thử tối đa 3 vòng, nếu vẫn thất bại thì dừng lại và báo cáo điểm blocking”, tránh để nó tiêu tốn quá nhiều Token theo hướng sai.

### Hỏi đáp về codebase

Khi tiếp quản project lạ, trước hết tôi dùng Claude Code như một hướng dẫn viên tạm thời. Đừng vội để nó sửa file, hãy hỏi call chain trước:

```text
Flow đầy đủ của user login là gì? Từ lúc HTTP request đi vào đến khi ghi session,
hãy liệt kê các class và method liên quan, không sửa file.
```

Hoặc:

```text
State machine của order trong project này được định nghĩa ở đâu? Mỗi state chuyển tiếp như thế nào?
Nếu có constraint ngầm thì cũng hãy chỉ ra.
```

Tuy nhiên nội dung nó tổng hợp vẫn cần được kiểm tra ngẫu nhiên. Các đoạn gọi xuyên service, config switch và logic tương thích lịch sử là những nơi nó dễ trình bày rất trôi chảy nhưng thực tế lại bỏ sót một branch.

### Sửa bug cần cung cấp thông tin lỗi

Điều đáng sợ nhất khi sửa bug là chỉ ném cho nó một câu:

```text
Login có bug, giúp tôi sửa.
```

Về cơ bản đây là yêu cầu Claude đoán. Dán tài liệu gốc vào sẽ ổn định hơn nhiều:

```text
Dưới đây là log lỗi production, bước reproduce và request parameter liên quan.
Trước tiên hãy xác định nguyên nhân có thể xảy ra, đừng lập tức sửa code.
Sau khi tìm được root cause, bổ sung một test có thể reproduce rồi sửa lỗi.
```

Log, stack trace, thảo luận trên Slack, output Docker và kết quả test thất bại đều hữu ích hơn việc bạn kể lại “hình như là vấn đề cache”. Càng kể lại nhiều, Claude càng dễ bị dẫn lệch bởi phỏng đoán của bạn.

### Nhiều instance và Worktree

Đừng để một Claude làm tất cả. Các task độc lập có thể tách sang các session khác nhau để chạy song song.

Claude Code hỗ trợ dùng Git Worktree để cô lập các session khác nhau:

```bash
claude --worktree feature-auth
claude --worktree bugfix-payment
```

`--worktree` là argument được Claude Code chính thức hỗ trợ, sẽ tạo worktree cô lập bên dưới repository và khởi động session; directory mặc định là `.claude/worktrees/<name>/`, branch thường là `worktree-<name>`. Nếu muốn tự kiểm soát hoàn toàn directory và branch, cũng có thể tạo trước bằng command native của Git:

```bash
git worktree add ../project-auth -b feature-auth
cd ../project-auth
claude
```

Mỗi Worktree có directory và branch độc lập. Một session sửa module authentication, session khác sửa payment bug, các file sẽ không giẫm lên nhau. Desktop app chính thức cũng tự tạo Worktree cho session mới, cùng hướng với CLI.

![Claude Code Git Worktree](https://oss.javaguide.cn/github/javaguide/ai/coding/claude-code-git-worktree.png)

Nếu đã có nhiều session chạy nền, có thể dùng:

```bash
claude agents
```

Agent View sẽ đặt các session nền vào một interface, cho biết session nào đang chạy, session nào cần bạn xác nhận và session nào đã hoàn thành. Sau khi dùng multi-session một thời gian, cách này gọn hơn nhiều so với mở một dãy cửa sổ terminal.

Có một điểm dễ mắc ở đây: trước khi bắt đầu sửa file, session nền sẽ tự chuyển mình vào một worktree độc lập dưới `.claude/worktrees/`, tránh việc nhiều session giẫm lên cùng một workspace. Nhưng nếu project có nhiều artifact sinh ra, hoặc pre-commit hook rất nhạy với path, việc liên tục cô lập lại trở thành gánh nặng. Trong trường hợp này có thể đặt `worktree.bgIsolation` thành `"none"` trong `.claude/settings.json` (cần v2.1.143+), để session nền sửa trực tiếp workspace. Đổi lại, các session chạy đồng thời có khả năng giẫm lên nhau, cần cân nhắc tùy project.

Nếu dùng `.worktreeinclude` để copy các file bị gitignore như `.env`, `.env.local`, `config/secrets.json` sang worktree mới, nhất định phải xác nhận chúng chỉ chứa credential local cho development, không phải credential production. Worktree cô lập thay đổi file, không có nghĩa là cô lập rủi ro secret.

### Đừng nhồi Commit và PR quá lớn trong một lần

Đừng để Claude commit một đống thay đổi lớn trong một lần.

Tôi thiên về việc tách thành các commit nhỏ. Mỗi commit chỉ làm một việc; trước khi commit, yêu cầu Claude đưa ra tóm tắt diff, command verify và rủi ro còn lại, sau đó tự xem lại `git diff --stat` và `git diff` của các file quan trọng.

Claude viết commit message và mô tả PR rất nhanh, nhưng cuối cùng đừng chỉ xem phần tổng kết của nó. Nó nói “chỉ sửa logic authentication” không đáng tin bằng việc bạn tự xem diff một lần. Mô tả PR chỉ cần viết rõ ba điều: đã sửa gì, đã test thế nào và còn điểm nào chưa xử lý hoàn toàn.

## Command thường dùng

Không cần học thuộc command, khi dùng thật chỉ cần gõ `/` để xem. Tôi thường ghi nhớ theo hai nhóm.

Nhóm đầu là command cơ bản. `/help` xem environment hiện tại thực sự có những command nào; `/diff` xem Claude đã sửa file và dòng nào; khi task dài chạy chậm hoặc bắt đầu lệch, trước tiên xem `/context`, khi context quá đầy thì dùng `/compact`; quyền hạn xem ở `/permissions`, vị trí cấu hình rule và memory xem ở `/memory`, kết nối MCP xem ở `/mcp`, phân tách usage xem ở `/usage`.

Nhóm thứ hai là command liên quan đến bundled skills / workflow. `/code-review` dùng để quét correctness bug, điều kiện biên và rủi ro tiềm ẩn trong các thay đổi hiện tại, có thể chỉ định effort, chẳng hạn `/code-review high`; thêm `--comment` có thể gửi phát hiện thành inline comment trên GitHub PR; thêm `--fix` sẽ áp dụng review findings vào workspace.

`/simplify` hiện phù hợp hơn khi xem là cleanup-only review, dùng để xử lý các mục cleanup như tái sử dụng, đơn giản hóa và hiệu quả, đồng thời tự động áp dụng fix. Nó không phải bug-hunting review đầy đủ. Quan hệ giữa `/simplify` và `/code-review --fix` đã thay đổi trong các version cũ; nếu hành vi command bạn thấy không giống bài này, hãy ưu tiên xem `/help` hiện tại và tài liệu commands chính thức.

`/batch` dùng cho thay đổi lớn trên nhiều module với ranh giới rõ ràng; nó sẽ tách yêu cầu thành nhiều work unit, mở Worker chạy nền và thực hiện song song trong worktree cô lập. `/loop` dùng để lặp lại việc thực thi Prompt theo khoảng thời gian, phù hợp để polling CI hoặc bảo trì định kỳ; khi cần liên tục sửa ngay cho đến khi test pass toàn bộ, migration hoàn tất hay đạt điều kiện tương tự, hãy dùng `/goal` và viết rõ tiêu chí nghiệm thu cùng điều kiện dừng. `/run` dùng để khởi động application và xem thay đổi có hiệu lực hay không; `/verify` nhẹ hơn, chủ yếu thực hiện build và verify runtime, nhanh chóng xác nhận có vấn đề compile hoặc runtime hay không.

Các command và bundled skills này thay đổi rất nhanh; danh sách nhìn thấy có thể khác nhau giữa version, platform và gói dịch vụ. Viết bài có thể giới thiệu kinh nghiệm, nhưng khi dùng trên máy của mình vẫn nên xem `/help` và tài liệu commands chính thức trước. Hành vi của một version không nhất thiết sẽ giữ nguyên lâu dài.

`/compact` còn có một điểm dễ bỏ qua: sau khi compress, một số rule sẽ không lập tức quay lại context. `CLAUDE.md` ở root sẽ được inject lại, nhưng nested rule trong subdirectory chưa chắc trở lại ngay. Sau khi compress task dài, tốt nhất hãy để Claude nhắc lại mục tiêu hiện tại, file đã sửa, rủi ro còn lại và command verify tiếp theo trước khi tiếp tục.

Tôi đã viết chi tiết command trong bài [Giải thích chi tiết command cốt lõi của Claude Code: code-review, loop, goal, batch, run, verify](https://javaguide.cn/ai-coding/practices/claudecode-commands.html), nên ở đây không lặp lại quá dài.

## Cách viết prompt

### Dùng tiếng Anh và tiếng Việt ở đúng chỗ

Trong task lập trình, tiếng Anh thường ổn định hơn. Tôi không muốn nâng điều này thành “tiếng Việt không được”, chỉ là code, tên library, thông tin lỗi và tài liệu API vốn đã dùng rất nhiều tiếng Anh. Những từ như `modal`, `debounce`, `retry policy`, `transaction boundary`, nếu cố dịch cứng sang tiếng Việt lại dễ mất nghĩa.

Nhưng bối cảnh nghiệp vụ, rule sản phẩm và copy tiếng Việt đương nhiên vẫn chính xác hơn khi viết bằng tiếng Việt. Thói quen của tôi là: action code, technical constraint và thuật ngữ cố gắng dùng tiếng Anh; semantic nghiệp vụ, tiêu chí nghiệm thu và copy tiếng Việt thì nói rõ bằng tiếng Việt.

### Giới hạn phạm vi

Cũng cần cẩn thận với câu “hãy điều tra project này”. Claude sẽ rất nghiêm túc tìm file khắp nơi, đọc một lúc thì context bị lấp đầy.

Có thể viết như sau:

```text
Chỉ điều tra hai directory src/payment và src/order.
Mục tiêu là xác định sau khi thanh toán order thành công thì việc trừ inventory được trigger ở đâu.
Không sửa file, chỉ liệt kê call chain và các class liên quan.
```

Sau khi viết rõ phạm vi, mục tiêu và action bị cấm, phạm vi tìm file và sửa code của nó sẽ thu hẹp đáng kể.

### Đưa ví dụ chuẩn

Hãy để Claude viết code theo style của project, đừng chỉ nói “tham khảo best practice”. Phạm vi này quá rộng, nó rất dễ viết ra một bộ trông có vẻ tốt nhưng hoàn toàn không phù hợp với project của bạn.

Đưa cho nó một mẫu hiện có thường hiệu quả hơn:

```text
Đọc UserController.java, UserService.java và UserDTO.java.
Tham khảo cách phân tầng, constructor injection, format trả về Result<T> và xử lý exception của chúng.
Bổ sung một OrderController cho việc query order, không đưa vào một response structure mới.
```

Style sẵn có trong project thường có tính ràng buộc hơn bộ “best practice” bên ngoài. Đặc biệt với project cũ, phân tầng, response structure, xử lý exception và format log thường mang theo gánh nặng lịch sử cùng thói quen của team. Hãy để Claude đọc mẫu trước rồi bổ sung theo đó, output sẽ sát với repository hiện tại hơn.

### Với frontend, đừng chỉ nói “làm cho đẹp”

Nếu để Claude viết frontend, đừng chỉ nói “hiện đại, đơn giản, cao cấp”.

Các từ kiểu này quá trống, cuối cùng rất dễ nhận được một tổ hợp quen thuộc: font Inter, gradient màu tím, card bo tròn lớn và cảm giác landing page marketing toàn màn hình. Hệ thống quản trị đặc biệt dễ thất bại; vốn là page được nhân viên vận hành dùng với tần suất cao nhưng cuối cùng lại bị làm thành website sản phẩm.

Thông thường tôi sẽ viết cứng hơn một chút:

```text
Dùng component Ant Design hiện có, không thêm UI library.
Page là công cụ vận hành backend, ưu tiên mật độ thông tin, không dùng phong cách landing page marketing.
Giữ màu chủ đạo theo CSS variable của project, không thêm background gradient màu tím.
Tham khảo khu vực filter và layout table của src/pages/UserList.tsx.
```

Quy chuẩn design cũng có thể làm thành Skill để trước mỗi lần viết frontend Claude đọc constraint thị giác của project. Trước tiên hãy chặn những khuôn mẫu không nên xuất hiện. Công cụ backend hãy giữ đúng là công cụ backend; mật độ thông tin, khả năng scan và phản hồi thao tác quan trọng hơn “cảm giác không khí”.

Bài [Danh sách lựa chọn Skills cho AI coding](https://javaguide.cn/ai-coding/practices/programmer-essential-skills.html) cũng có đề xuất các Skill open source liên quan đến frontend.

## Các pattern thất bại thường gặp

Điều tôi gặp nhiều nhất là session quá lẫn lộn. Trong một session đồng thời trao đổi requirement, debug, refactor và release, Claude nhanh chóng bắt đầu mang quán tính của topic trước sang task tiếp theo. Khi chuyển task, hãy dùng `/clear` trực tiếp; khi cần thì viết một `HANDOFF.md`, đừng cố tiếp tục nói chuyện trong cùng context.

Loại thứ hai là vòng lặp sửa sai. Nếu cùng một lỗi đã sửa ba lần vẫn không đúng, đừng tiếp tục mài trong context cũ. Hãy dừng lại, viết lại prompt ban đầu, nói rõ mục tiêu, evidence và action bị cấm.

`CLAUDE.md` phình ra cũng rất thường gặp. Khi có quá nhiều rule mà Claude ngược lại không tuân thủ, đừng tiếp tục thêm rule; trước hết hãy xóa những nội dung có thể đọc ra từ code, chỉ giữ constraint đã tổng kết sau lỗi thực tế.

Một loại khác là điều tra không có ranh giới. Khi yêu cầu Claude “xem project này”, nó có thể đọc một lần vài trăm file và nhanh chóng dùng hết context. Hãy giới hạn directory, mục tiêu và action bị cấm, hoặc giao cho Sub-Agent.

Test pass toàn bộ cũng không có nghĩa hành vi đã đúng. Hãy để Claude đưa ra evidence, so sánh branch main và feature, cần xem log thì xem log, cần verify thủ công thì chạy verify thủ công.

Quyền quá rộng là nguy hiểm nhất. Vì muốn bớt xác nhận mà bypass trực tiếp, về sau sẽ rất khó điều tra thao tác nhầm. Hãy cấu hình allow/deny, Auto Mode, cô lập bằng container và credential tạm thời theo từng tầng rủi ro.

## Tổng kết

Lúc đầu rất dễ chỉ tập trung vào việc “để nó viết thêm nhiều code”. Dùng lâu mới nhận ra thứ ảnh hưởng đến kết quả lại là những thói quen engineering rất bình thường: viết rõ quy tắc project trong `CLAUDE.md`, task phức tạp thì plan trước, thay đổi xong bắt buộc verify, điều tra dài thì giao cho Sub-Agent, nhiều task thì cô lập bằng Worktree, đừng mở quyền quá rộng ngay một lần.

Trong project thực tế, có thể bắt đầu từ một directory, một module và một chuỗi task có thể verify; để Claude hoàn thành ổn định trong phạm vi nhỏ rồi từng bước tăng độ phức tạp của task.
