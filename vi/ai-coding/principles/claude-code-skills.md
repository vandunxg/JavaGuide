---
title: Chi tiết triển khai kỹ thuật và cách vận hành Claude Code Skills
description: Từ cấu trúc file, cơ chế phát hiện và tải, Front Matter, context động, giới hạn bảo mật và cách phối hợp với Subagent của Claude Code Skills, giải thích cách Skills biến workflow có thể tái sử dụng thành năng lực Agent được tải theo nhu cầu.
category: Nguyên lý AI coding
tag:
  - Claude Code
  - Skills
  - AI Agent
  - AI coding
head:
  - - meta
    - name: keywords
      content: Claude Code,Skills,Agent Skills,SKILL.md,Front Matter,context động,Subagent,Plugin,AI coding
---

Nhiều độc giả phản hồi rằng Skills thường xuất hiện trong các buổi phỏng vấn hiện nay, nên sau hai bài đã viết trước đó, tôi lại cố gắng viết thêm một bài.

Dưới đây là nội dung chính.

Bạn còn nhớ thời gian mới dùng Claude Code, tôi rất dễ nhét mọi loại quy tắc vào `CLAUDE.md`.

Code style, quy ước thư mục, lệnh test, những thứ này đưa vào không có vấn đề. Nhưng sau đó một số checklist code review, workflow tổng hợp PR, các bước nghiệm thu UI cũng bắt đầu bị chất vào đó.

Vấn đề xuất hiện từ đây.

Những workflow này thực sự hữu ích, nhưng không phải vòng task nào cũng cần dùng. Nếu lần nào cũng đưa vào, chúng sẽ thêm rất nhiều thông tin vô ích, thậm chí gây nhiễu phán đoán của model.

Những nội dung như vậy không nên tiếp tục nhét vào `CLAUDE.md`. Nếu bạn luôn phải copy cùng một đoạn instructions, checklist hoặc workflow nhiều bước vào cuộc hội thoại, hoặc một mục nào đó trong `CLAUDE.md` đã giống một sổ tay thao tác, bạn có thể tách nó ra.

Khác biệt chủ yếu nằm ở cách tải. `CLAUDE.md` thường được tải làm context cố định khi bắt đầu session; Skill bình thường chỉ hiển thị tên và mô tả, đến khi thực sự khớp mới tải nội dung đầy đủ. Tài liệu tham khảo dài, checklist và hướng dẫn script không cần chen vào context ngay từ đầu.

Bài viết này chủ yếu nói về triển khai kỹ thuật và cách vận hành Claude Code Skills. Tôi sẽ tham khảo các tài liệu phân tích source code của cộng đồng để xem chi tiết triển khai, nhưng cách dùng hiện tại sẽ lấy tài liệu chính thức và changelog làm chuẩn.

Nếu muốn tìm hiểu có hệ thống về sự khác nhau giữa Agent Skills với Prompt, MCP và Function Calling trước, bạn có thể xem bài [Agent Skills là gì? Khác Prompt, MCP ở đâu?](https://javaguide.cn/ai/agent/skills.html) tôi đã viết trước đây. Nếu quan tâm hơn đến những Skill có sẵn đáng cài, bạn có thể xem trực tiếp [Các Skills thiết yếu cho AI coding: TDD, code review, tự động hóa web và thực chiến MCP](https://javaguide.cn/ai-coding/programmer-essential-skills.html).

## Skills giải quyết vấn đề gì

Trước tiên hãy xem cách phân công giữa `CLAUDE.md` và Skill.

`CLAUDE.md` phù hợp để chứa các sự thật về project và quy tắc dài hạn cần dùng trong mọi vòng, chẳng hạn code style, quy ước thư mục, lệnh thường dùng và mô tả kiến trúc.

Skill phù hợp để chứa workflow có context kích hoạt rõ ràng. Chúng cần được tái sử dụng, nhưng không cần đi cùng lúc khởi động session.

Điển hình nhất là:

- Một bộ checklist code review;
- Một bộ bước xử lý sự cố production;
- Một workflow tạo bản tổng hợp PR;
- Một design guideline chỉ cần khi sửa UI;
- Một workflow TDD chỉ dùng khi viết test.

Điểm chung của chúng là: các bước cố định, nội dung không ngắn, nhưng chỉ xuất hiện trong task cụ thể. Nếu viết toàn bộ vào `CLAUDE.md`, chúng sẽ trở thành chi phí context bổ sung ngay lúc khởi động, càng chất nhiều càng nặng.

Bạn có thể hiểu Skill là một sổ tay thao tác mở theo nhu cầu: bình thường chỉ cho Claude biết có năng lực này, đến khi thực sự dùng mới lấy hướng dẫn đầy đủ ra.

![So sánh Skill với Prompt, MCP và Function Calling](https://oss.javaguide.cn/github/javaguide/ai/skills/skill-prompt-function-calling-mcp-comparison.webp)

`CLAUDE.md` thì ngược lại. Tài liệu chính thức khuyến nghị để nó chứa những nội dung cần biết trong mọi vòng, chẳng hạn lệnh build, quy ước project, cấu trúc thư mục và các quy tắc bắt buộc phải luôn tuân thủ.

Nếu một đoạn nội dung đã là workflow nhiều bước, hoặc chỉ ảnh hưởng đến một phần cục bộ nào đó trong codebase, nó sẽ phù hợp hơn khi chuyển sang Skill hoặc path-scoped rule.

Khi thực sự tách trong project, tôi thường không băn khoăn về tên trước, mà xem đoạn nội dung đó đang mắc ở đâu.

- Nếu mắc ở chỗ “quy tắc phải có hiệu lực trong mọi vòng”, đó giống vấn đề của `CLAUDE.md` hơn. Nếu mắc ở chỗ “một workflow bị copy đi copy lại”, đó giống vấn đề của Skill hơn.
- Nếu mắc ở chỗ “task quá dài, toàn bộ quá trình sẽ chiếm context của session chính”, mới cân nhắc Subagent. Nếu mắc ở chỗ “mọi người trong team đều phải cài một bộ”, mới cân nhắc Plugin.

Tôi dùng một bảng để tổng hợp các khái niệm đã nói ở trên:

| Cơ chế      | Vấn đề chính được giải quyết                                  |
| ----------- | ------------------------------------------------------------- |
| `CLAUDE.md` | Quy tắc project thường trực và quy ước dài hạn                |
| Skill       | Workflow và checklist chỉ dùng trong task cụ thể              |
| Subagent    | Ủy thác task dài hoặc task nhánh cho Agent khác               |
| Plugin      | Phân phối các năng lực mở rộng như Skills, Agents, Hooks, MCP |

Skill trong Claude Code có thể hiểu là “prompt-based command”.

Phần custom command cũng đã được hợp nhất vào hệ thống Skills. Hiện tại `.claude/commands/deploy.md` và `.claude/skills/deploy/SKILL.md` đều có thể tạo `/deploy`; `.claude/commands/` cũ chưa cần di chuyển ngay và vẫn tương thích.

Subagent giải quyết câu hỏi “ai làm”; Skill giải quyết câu hỏi “làm thế nào”.

Plugin chịu trách nhiệm phân phối. Một Plugin có thể mang theo Skills, Agents, Hooks và MCP Servers. Nếu doanh nghiệp hoặc team muốn phát hành năng lực thống nhất, Plugin phù hợp hơn việc sao chép riêng từng file Skill.

Nếu trong project đồng thời có `CLAUDE.md`, `AGENTS.md`, rule cục bộ, SPEC và Skills, cũng có thể tách theo cách này: rule thường trực đặt trong file rule, workflow có thể tái sử dụng giao cho Skill, tiêu chuẩn nghiệm thu của task hiện tại đặt vào SPEC.

![Cách phân công giữa CLAUDE.md và các file rule khác](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/claude-md-best-practices-rule-files-relationship.png)

Nội dung phù hợp để biến thành Skill thường có một số đặc điểm: thường xuyên tái sử dụng, có context kích hoạt rõ ràng, các bước tương đối cố định, nội dung tương đối dài, không phù hợp để thường trực trong context, tốt nhất còn có thể đi kèm supporting files hoặc script (ví dụ `scripts/`, `references/`, `templates/`).

Code review, TDD, tổng hợp PR, kiểm tra thay đổi database, nghiệm thu UI và kiểm tra log đều thuộc nhóm task này.

Nội dung không phù hợp để làm Skill là hard rule luôn phải tuân thủ trong project. Ví dụ “toàn bộ Java code phải dùng Google Java Style” phù hợp hơn khi đặt trong `CLAUDE.md` hoặc rule của project.

Giới thiệu chi tiết và best practice về `CLAUDE.md` có thể xem bài [Best practice của CLAUDE.md: nên viết gì, không nên viết gì và tách thế nào khi project lớn lên](https://javaguide.cn/ai-coding/practices/claude-md-best-practices.html) tôi đã viết.

## Cách viết `SKILL.md`

Một Skill dựa trên file system thường có cấu trúc thư mục như sau:

```text
.claude/skills/
  pr-summary/
    SKILL.md
    scripts/
      collect-pr-info.sh
    references/
      review-checklist.md
```

`SKILL.md` gồm hai phần:

1. YAML frontmatter: mô tả metadata như tên, điều kiện kích hoạt, quyền tool, model và context thực thi.
2. Markdown body: hướng dẫn thao tác thực sự gửi cho Claude.

Một ví dụ tối giản:

```md
---
name: pr-summary
description: Summarize a pull request and list key risks
allowed-tools: Bash(gh *)
---

Read the pull request diff and comments, then summarize:

1. Main changes
2. Risky files
3. Missing tests
4. Suggested follow-up
```

Khi thực thi `/pr-summary`, Claude Code sẽ render Skill này thành prompt rồi giao cho model.

`parseSkillFrontmatterFields()` trong source code hỗ trợ khá nhiều field, trước tiên có thể xem các field thường gặp dưới đây:

| Field                      | Tác dụng                                                      |
| -------------------------- | ------------------------------------------------------------- |
| `name`                     | Tên hiển thị; tên thư mục thường quyết định tên command       |
| `description`              | Để model phán đoán khi nào nên dùng                           |
| `when_to_use`              | Mô tả trigger chi tiết hơn                                    |
| `allowed-tools`            | Pre-approve các tool Skill đó được dùng                       |
| `model`                    | Chỉ định alias của model                                      |
| `effort`                   | Chỉ định mức độ suy luận/nỗ lực                               |
| `user-invocable`           | Có cho phép user gọi trực tiếp qua `/skill-name` hay không    |
| `disable-model-invocation` | Cấm model tự gọi, chỉ cho phép user gọi thủ công              |
| `paths`                    | Path kích hoạt có điều kiện                                   |
| `context`                  | Hỗ trợ `fork`, cho phép Skill chạy trong context của subagent |
| `agent`                    | Gắn với Agent được chỉ định                                   |
| `shell`                    | Chỉ định command lấy context động dùng bash hay powershell    |

Đừng ngay từ đầu chất toàn bộ field vào đây. Phần lớn Skill chỉ cần `description`, `allowed-tools` và hướng dẫn trong body. Càng nhiều field, chi phí bảo trì càng cao.

Có một số field dễ nhầm:

| Field           | Phù hợp hơn để giải quyết vấn đề gì                                 |
| --------------- | ------------------------------------------------------------------- |
| `allowed-tools` | Thu hẹp phạm vi tool mà Skill hiện tại có thể dùng trực tiếp        |
| `context: fork` | Cho workflow dài, task nghiên cứu và review chạy trong context fork |
| `agent`         | Chỉ định Agent nào thực thi Skill này                               |

Ví dụ:

```yaml
context: fork
agent: Explore
allowed-tools: Bash(gh *)
```

Loại cấu hình này phù hợp với các task như tổng hợp PR, review module và tổng hợp tài liệu. Session chính không nhất thiết phải ghi nhớ toàn bộ quá trình, chỉ cần kết quả là đủ.

Skills cũng hỗ trợ thay thế tham số. Cách đơn giản nhất là `$ARGUMENTS`:

```md
---
name: fix-issue
description: Fix a GitHub issue
disable-model-invocation: true
---

Fix GitHub issue $ARGUMENTS following our coding standards.
```

Thực thi:

```bash
/fix-issue 123
```

Trong nội dung Claude nhận được, `$ARGUMENTS` sẽ được thay bằng `123`.

Nếu muốn lấy tham số theo vị trí, có thể dùng `$ARGUMENTS[0]`, cũng có thể viết ngắn là `$0`:

```md
Migrate the $0 component from $1 to $2.
```

Thực thi:

```bash
/migrate-component SearchBar React Vue
```

`$0`, `$1`, `$2` lần lượt được thay bằng `SearchBar`, `React`, `Vue`.

## Claude Code phát hiện Skills như thế nào

Claude Code tải Skills từ nhiều nguồn. Các vị trí thường gặp gồm:

```text
~/.claude/skills/
.claude/skills/
```

Skill cấp user đặt trong `~/.claude/skills/`, tất cả project đều có thể dùng. Skill cấp project đặt trong `.claude/skills/`, phù hợp để chia sẻ với team.

![Ví dụ thư mục .claude/skills trong project](https://oss.javaguide.cn/github/javaguide/ai/coding/claude-code-project-skills-folder.png)

Theo source code, thư mục Skills dùng cấu trúc:

```text
skill-name/SKILL.md
```

Nói cách khác, một file `.md` đứng riêng dưới thư mục `/skills/` không phải format Skill tiêu chuẩn; trong thư mục phải có `SKILL.md`.

Nguồn Skill của Claude Code nhìn chung có thể chia thành vài loại:

| Loại              | Nguồn                           | Mô tả                                      |
| ----------------- | ------------------------------- | ------------------------------------------ |
| Skill cấp user    | `~/.claude/skills/`             | Tái sử dụng cá nhân dài hạn                |
| Skill cấp project | `.claude/skills/`               | Chia sẻ trong project hoặc team            |
| Managed Skills    | Thư mục theo policy quản trị    | Tổ chức phân phối thống nhất               |
| Bundled Skills    | Tích hợp sẵn trong Claude Code  | Ví dụ `/code-review`, `/debug`, `/loop`... |
| Plugin Skills     | Plugin cung cấp                 | Cài đặt và bật cùng plugin                 |
| MCP Skills        | Năng lực được MCP Server ánh xạ | Đến từ MCP Server                          |

Claude Code có một số bundled skills, chẳng hạn `/code-review`, `/batch`, `/debug`, `/loop` và `/claude-api`. Chúng khác với command tích hợp thông thường, thuộc loại prompt-based skill.

![Mô tả Bundled skills trong tài liệu chính thức của Claude Code](https://oss.javaguide.cn/github/javaguide/ai/coding/claude-code-bundled-skills-docs.png)

Cũng cần chú ý đến các thư mục `.claude/skills` lồng nhau.

Sau v2.1.178, các thư mục `.claude/skills` lồng nhau cũng được tải khi xử lý file tương ứng. Khi xảy ra xung đột tên, Skill lồng nhau sẽ xuất hiện dưới dạng `<dir>:<name>`, tránh ghi đè Skill cùng tên ở bên ngoài.

Điều này gần với tư duy của rule project: **cấu hình càng gần working directory hiện tại càng thể hiện rõ context cục bộ.**

Tuy nhiên, không nên lạm dụng Skill lồng nhau. Chỉ tách khi thư mục con thực sự có workflow độc lập, ví dụ:

- `frontend/.claude/skills/ui-review/SKILL.md`
- `backend/.claude/skills/api-contract-check/SKILL.md`
- `docs/.claude/skills/article-review/SKILL.md`

Nếu chỉ để phân loại, thư mục thông thường và tên file là đủ.

`.claude/commands/` phiên bản cũ vẫn tương thích. Trong source code cũng có thể thấy legacy commands loader: nếu thư mục command cũ tồn tại `SKILL.md`, nó sẽ được xử lý theo cách của Skill; nếu không thì tiếp tục tải dưới dạng Markdown command.

Tài liệu chính thức đã ghi rõ: custom commands đã được hợp nhất vào Skills, nhưng các file `.claude/commands/` hiện có vẫn tiếp tục hoạt động. Khi viết năng lực mới, khuyến nghị dùng trực tiếp `.claude/skills/<name>/SKILL.md`.

## Điều gì xảy ra sau khi Skill được gọi

Skill bình thường không đưa toàn bộ body vào context.

Claude Code chủ yếu dùng tên, mô tả, `when_to_use` và các thông tin frontmatter khác của Skill để cho model biết những năng lực nào có thể dùng.

Trong source code còn có `estimateSkillFrontmatterTokens()`, chỉ ước tính token của name, description và whenToUse, vì nội dung đầy đủ chỉ được tải khi gọi.

Khi user thực thi `/skill-name`, hoặc model phán đoán một Skill phù hợp với task hiện tại, Claude Code mới gọi `getPromptForCommand()` để render Skill body.

Đây cũng là lý do chính khiến Skills tiết kiệm context hơn `CLAUDE.md` dài.

![Chuỗi thực thi Agent](https://oss.javaguide.cn/github/javaguide/ai/skills/skill-agent-execution-link.webp)

Sau khi Skill được gọi, Claude Code trước tiên nhận Markdown body, sau đó lần lượt làm vài việc:

1. Mở rộng tham số, chẳng hạn `$ARGUMENTS`, `$0`.
2. Thay thế `${CLAUDE_SKILL_DIR}`.
3. Thay thế `${CLAUDE_SESSION_ID}`.
4. Nếu không đến từ MCP, thực thi shell command nhúng.
5. Trả prompt cuối cùng cho model.

Phần triển khai tương ứng trong `createSkillCommand()` chính là `getPromptForCommand()`. Nó đợi đến lúc thực sự được gọi mới xử lý, không render sẵn toàn bộ Skill trong giai đoạn khởi động.

Skill có thể đi kèm supporting files, chẳng hạn script, tài liệu tham khảo và template. Cấu trúc thư mục có thể là:

```text
my-skill/
  SKILL.md
  scripts/
  references/
  templates/
```

Nếu body cần tham chiếu đến path của script, có thể dùng `${CLAUDE_SKILL_DIR}`:

```md
Run this helper:

!`${CLAUDE_SKILL_DIR}/scripts/collect-context.sh`
```

Nhờ vậy, sau khi di chuyển thư mục Skill cũng ít bị hỏng hơn.

Tuy nhiên, không nên đưa toàn bộ supporting files vào body. Cách viết tốt hơn là nói cho Claude biết trong `SKILL.md` khi nào cần đọc file nào. Dùng đến đâu thì đọc đến đó, không dùng thì đừng đưa vào context.

Đó chính là progressive disclosure: trước tiên cho model biết “có năng lực này”, sau khi khớp mới đọc body; trong body chỉ đặt khung workflow, còn tài liệu thực sự dài tiếp tục đặt trong supporting files.

![Progressive disclosure (mô hình ba lớp)](https://oss.javaguide.cn/github/javaguide/ai/skills/skills-progressive-disclosure-three-layer-model.png)

Vì vậy, `SKILL.md` không phù hợp để viết thành README siêu dài. Trong body nên ưu tiên viết khi nào dùng, thực hiện theo thứ tự nào, trường hợp nào không làm và xử lý dự phòng khi thất bại; checklist dài, template và hướng dẫn script đặt trong `references/`, `templates/`, `scripts/`.

![Nên giới hạn body SKILL.md trong 500 dòng](https://oss.javaguide.cn/github/javaguide/ai/skills/keep-skill-md-content-under-500-lines-for-best-performance.png)

Cũng có thể hiểu sự khác nhau giữa `CLAUDE.md` và Skill từ chiến lược tải:

| Nội dung                      | Phù hợp đặt ở đâu        |
| ----------------------------- | ------------------------ |
| Quy chuẩn coding của project  | `CLAUDE.md`              |
| Mô tả cấu trúc thư mục        | `CLAUDE.md`              |
| Workflow code review          | Skill                    |
| Workflow tổng hợp PR          | Skill                    |
| UI nghiệm thu checklist       | Skill                    |
| Script xử lý sự cố production | Skill + supporting files |

Đừng xem Skill như một `CLAUDE.md` dài hơn. Giá trị của Skill nằm ở việc tải theo nhu cầu, không phải đổi sang thư mục khác để tiếp tục chất rule.

## Context động và giới hạn bảo mật

Skills hỗ trợ inject context động, cú pháp là:

```md
Trạng thái Git hiện tại:
!`git status --short`
```

Cũng hỗ trợ dạng code block:

````md
```!
git log --oneline -5
```
````

Các command này được thực thi trước khi nội dung Skill gửi cho Claude. Output của command sẽ thay thế placeholder ban đầu, model nhìn thấy kết quả cuối cùng chứ không phải bản thân command.

Tài liệu chính thức cũng nhấn mạnh: đây là preprocessing. Claude chỉ nhìn thấy prompt đã render.

Điều này rất khác với việc Claude gọi Bash.

Claude gọi Bash là model quyết định dùng tool trong agent loop. Nó tạo ra một tool call, kết quả tool đi vào lịch sử hội thoại.

Command động trong Skill là prompt preprocessing. Command được thực thi trước, output được chèn vào Skill prompt. Model sẽ không thấy quá trình “tôi chuẩn bị thực thi command này”.

Nội dung phù hợp để đặt trong context động thường là thông tin context ổn định, chỉ đọc, rủi ro thấp, chẳng hạn:

- `git status --short`
- `git diff --name-only`
- `gh pr view --comments`
- Script chỉ đọc có sẵn trong project

Đừng viết command sửa file, commit code hoặc xóa resource vào context động. Context động nên chịu trách nhiệm “thu thập tài liệu”, không chịu trách nhiệm “thực hiện thay đổi”.

Skill từ nguồn MCP đặc biệt hơn. Điều kiện phán đoán trong source code là: chỉ khi `loadedFrom !== 'mcp'` mới thực thi shell nhúng.

MCP Skill đến từ MCP Server từ xa và không nhất thiết đáng tin. Nếu cho phép server từ xa trả về một Skill có command động rồi thực thi trên máy local, điều đó sẽ trở thành rủi ro remote code execution.

Vì vậy, Skill từ nguồn MCP sẽ bỏ qua shell nhúng. Chỉ Skill từ file system, project local và nguồn đáng tin cậy mới đi theo pipeline preprocessing này.

Đồng thời, ngay cả Skill local cũng sẽ đi qua cùng một bộ kiểm tra quyền tool trước khi thực thi command. `allowed-tools` có thể cấp phép một phần command cho Skill hiện tại, nhưng không phải thực thi vô điều kiện.

Skill bên thứ ba cũng cần được kiểm tra theo cách này. Trước khi cài, ít nhất hãy xem qua `SKILL.md`, `scripts/`, `references/`, xác nhận bên trong không có command nguy hiểm, script bất thường hoặc quyền quá rộng. Cài Skill đồng nghĩa giao một workflow cho Agent thực thi; khi nguồn không rõ ràng, đừng vội cho nó vào project.

Trong môi trường doanh nghiệp, tài liệu settings chính thức có một cấu hình liên quan đến governance: `strictPluginOnlyCustomization`.

Nó có thể giới hạn nguồn của skills, agents, hooks và MCP servers. Ví dụ:

```json
{
  "strictPluginOnlyCustomization": ["skills", "hooks"]
}
```

Sau khi bị khóa, các nguồn cấp user và cấp project sẽ bị bỏ qua; chỉ tải năng lực do plugin cung cấp, do managed settings cung cấp hoặc năng lực tích hợp sẵn.

Loại cấu hình này phù hợp với môi trường team hoặc doanh nghiệp. Project cá nhân thường không cần dùng, nhưng nếu công ty muốn quản lý thống nhất nguồn mở rộng của công cụ AI coding thì không thể chỉ dựa vào thỏa thuận miệng.

## Skills phối hợp với Agent như thế nào

Skill có thể chạy trong Subagent.

Trong tài liệu chính thức có phần hướng dẫn “Run skills in a subagent”, frontmatter của Skill cũng hỗ trợ `context: fork` và `agent`.

Ví dụ, một Skill tổng hợp PR có thể cho Explore agent chạy trong context fork:

```yaml
context: fork
agent: Explore
allowed-tools: Bash(gh *)
```

Như vậy session chính không cần tự ghi nhớ toàn bộ PR diff, comment và danh sách file, chỉ cần nhận kết quả tổng hợp.

`context: fork` phù hợp với ba loại tình huống:

1. Quá trình của Skill rất dài;
2. Skill cần đọc nhiều file hoặc thông tin bên ngoài;
3. Session chính chỉ quan tâm kết quả, không quan tâm toàn bộ quá trình.

Ví dụ như tạo bản tóm tắt rủi ro PR, review chỉ đọc một module, tổng hợp tài liệu và issue hoặc tạo kế hoạch migration đều có thể cân nhắc fork.

Những task không phù hợp để đưa vào fork là task bắt buộc phải tương tác liên tục với session chính. Ví dụ bạn đang điều chỉnh thủ công một design cốt lõi, mỗi bước của Skill đều cần bạn xác nhận, vậy thì không nên fork nó ra ngoài.

Agent Teams cũng tạo thêm chi phí context. Tài liệu chính thức về chi phí từng nhắc rằng teammates sẽ tự động tải `CLAUDE.md`, MCP servers và Skills. Nói cách khác, Agent Teams không chỉ thêm vài prompt; mỗi teammate đều có chi phí khởi động riêng.

Điều này không có nghĩa là không nên dùng Skills, mà là cần kiểm soát mô tả và phạm vi kích hoạt của Skill:

- Viết `description` rõ ràng, không để model kích hoạt nhầm;
- Đặt nội dung dài trong supporting files, không nhét toàn bộ vào `SKILL.md`;
- Dùng `disable-model-invocation` để giới hạn Skill chỉ được gọi thủ công;
- Project team lớn dùng `strictPluginOnlyCustomization` để kiểm soát nguồn.

Mục tiêu thiết kế của Skill là tải theo nhu cầu. Nếu mô tả quá chung chung, kích hoạt quá thường xuyên, nó sẽ biến từ “tiết kiệm context” thành “chi phí bổ sung”.

Trong project thực tế, tôi thường tách theo quy tắc dưới đây:

| Nội dung                               | Đặt ở đâu                              |
| -------------------------------------- | -------------------------------------- |
| Quy tắc phải tuân thủ trong mọi vòng   | `CLAUDE.md`                            |
| Rule chỉ có hiệu lực trong path cụ thể | `.claude/rules/` hoặc Skill có `paths` |
| Workflow thao tác có thể tái sử dụng   | Skill                                  |
| Tài liệu tham khảo dài                 | Supporting files của Skill             |
| Nhánh tìm kiếm, review và verification | Subagent                               |
| Task phối hợp nhiều vai trò            | Agent Teams                            |

Ví dụ: bạn cần refactor một backend interface.

Đặt coding convention và lệnh test của project trong `CLAUDE.md`; đặt workflow kiểm tra tính tương thích interface trong Skill `api-contract-check`; đặt checklist review trong Skill `code-review`; giao việc tìm các caller cũ cho Subagent; nếu frontend, backend và test cần tiến hành song song thì cân nhắc Agent Teams.

Cách tách này giúp rule, workflow và executor mỗi phần đều rõ ràng. Đừng nhét tất cả vào session chính, cũng đừng mở Agent khắp nơi chỉ để tỏ ra cao cấp.

## Tổng kết

Tôi thích xem Skill như một **sổ tay thao tác được tải theo nhu cầu**.

Những gì phải tuân thủ trong mọi vòng tiếp tục đặt trong file rule; workflow chỉ dùng cho task cụ thể thì tách thành Skill; checklist, template và hướng dẫn script dài trong workflow thì tiếp tục tách vào supporting files.

Tiêu chuẩn phán đoán cũng rất đơn giản: nếu bạn đã bắt đầu copy đi copy lại cùng một đoạn prompt, hoặc một mục nào đó trong `CLAUDE.md` dài đến mức đọc như sổ tay, rất có thể nó nên trở thành Skill.

Ngược lại, nếu chỉ là hard rule như code style, lệnh test và quy ước thư mục phải tuân thủ trong mọi vòng, đừng viết Skill chỉ vì muốn dùng Skill. Đặt chúng trong `CLAUDE.md` sẽ trực tiếp hơn.
