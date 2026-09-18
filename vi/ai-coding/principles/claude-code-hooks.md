---
title: "Giải thích chi tiết Claude Code Hooks: lifecycle hook và workflow tự động hóa"
description: "Bắt đầu từ lifecycle của Claude Code, giải thích rõ thời điểm trigger của Hooks, các loại handler, input/output, chặn rủi ro, tự động format và nhắc thông báo, giúp bạn dùng Hooks biến các ràng buộc mềm trong prompt thành các hành động tự động hóa có thể audit và tái sử dụng."
category: AI Programming Principles
tag:
  - Claude Code
  - Hooks
  - AI Agent
  - AI Programming
head:
  - - meta
    - name: keywords
      content: Claude Code,Hooks,lifecycle hook,AI programming,automated workflow,PreToolUse,PostToolUse,UserPromptSubmit,SessionStart,permission control
---

Sau khi dùng Claude Code/ Codex để viết code đến một giai đoạn nhất định, nhiều người sẽ gặp cùng một vấn đề.

Vấn đề thường không nằm ở năng lực của model.

Ngược lại, chính là vì nó quá giỏi. Nó có thể sửa file, chạy command, tra cứu cấu trúc project, tạo script, cũng có thể xử lý một loạt task rất dài trong một lần. Vì vậy, bạn sẽ rất tự nhiên bắt đầu giao thêm nhiều hành động cho nó.

Rồi vấn đề xuất hiện: sau khi sửa file, lần này nó có quên format không? Khi chuẩn bị chạy Bash command, nó có vô tình kèm `rm -rf` không? Nó có tiện tay sửa vào `.env`, `.git/` hoặc config production không? Khi nó bị kẹt ở permission prompt, tôi có thể không cần liên tục nhìn terminal không?......

Những vấn đề này không phù hợp để chỉ giải quyết bằng prompt, vì ràng buộc của prompt không đủ và không thể bảo đảm hoàn toàn.

Lúc này Hooks trở nên cần thiết. Hooks giải quyết các vấn đề này bằng cách tự động trigger thực thi.

Khác biệt giữa hai cách có thể được tóm tắt trước bằng một hình:

![Prompt nhắc nhở phụ thuộc vào context và memory của model, còn điểm kiểm soát của Hooks bảo đảm hành động xảy ra bằng trigger tự động, audit script và chặn rủi ro](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/hooks-vs-prompts-guarantee.webp)

Tôi thích hiểu Hooks là các điểm kiểm soát cố định trong workflow của Claude Code. Khi session bắt đầu, user submit Prompt, trước và sau khi gọi tool, trước và sau khi nén context, đều có thể gắn các action xử lý tương ứng.

## Hooks thực chất là gì

Config Hook chủ yếu xoay quanh event và handler. Event quyết định khi nào trigger, handler nhận input tại thời điểm đó và hoàn thành action cụ thể.

Ví dụ, gắn một `command` handler vào `PostToolUse`: sau khi Claude sửa file thành công, Claude Code chuyển JSON của lần gọi tool này cho script, rồi script quyết định có chạy formatter hay không. Toàn bộ quá trình không cần Claude quay lại đọc prompt, đồng thời có thể lấy script ra để test riêng.

Handler cũng không giới hạn ở shell command. Official còn hỗ trợ HTTP endpoint, MCP tool và LLM prompt (xem mục "Hook handler fields" trong tài liệu chính thức [Hooks reference](https://code.claude.com/docs/en/hooks)).

![Năm loại handler do tài liệu chính thức Claude Code Hooks reference liệt kê: command, http, mcp_tool, prompt và agent](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/hooks-handler-types-official-docs.png)

Hình dưới đây đánh dấu các điểm trigger thường dùng:

![Claude Code Hooks tự động thực thi quanh các lifecycle node SessionStart, UserPromptSubmit, PreToolUse, PostToolUse, PermissionRequest và PreCompact](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/claude-code-hooks-lifecycle-map.webp)

Hook handler chủ yếu có năm loại:

| Loại       | Làm gì                                                   | Trường hợp phù hợp                                          |
| ---------- | -------------------------------------------------------- | ----------------------------------------------------------- |
| `command`  | Thực thi shell command                                   | format, log, chặn rủi ro, thông báo                         |
| `http`     | POST event JSON đến một URL                              | service audit của team, thông báo từ xa, policy tập trung   |
| `mcp_tool` | Gọi tool trên MCP server đã kết nối                      | Tái sử dụng năng lực MCP hiện có                            |
| `prompt`   | Dùng model phán đoán một lần và trả về JSON kiểu yes/no  | Phán đoán nhẹ, ví dụ kiểm tra task đã hoàn thành trước Stop |
| `agent`    | Khởi động subagent có khả năng truy cập tool để xác minh | Xác minh cần đọc file, tìm code, chạy command               |

Tuy nhiên, không phải event nào cũng dùng được cả năm loại handler này:

- `PreToolUse`, `PostToolUse`, `PermissionRequest`, `Stop` và các event khác hỗ trợ đủ cả năm loại;
- `Notification`, `PreCompact`, `ConfigChange` và các event khác không hỗ trợ `prompt` và `agent`;
- `SessionStart`, `Setup` chỉ hỗ trợ `command` và `mcp_tool`.

Trước khi dùng non-command handler, tốt nhất hãy kiểm tra trước phạm vi tương thích của event tương ứng.

Bước này có thể giao thẳng cho AI, bạn chỉ cần cung cấp bài viết này cho Coding Agent đang sử dụng hoặc gửi trực tiếp link tài liệu chính thức cho nó.

Các rule có thể viết thành script xác định thì ưu tiên giao cho `command`. Script có thể chạy độc lập ngoài Claude Code, nguyên nhân lỗi cũng dễ tái hiện hơn.

Chỉ khi quá trình xác minh cần đọc code, chạy test và phán đoán tổng hợp mới cần cân nhắc `prompt` hoặc `agent`. Trong đó, `agent` hooks vẫn được đánh dấu là experimental trong tài liệu chính thức, nên trước khi tích hợp cần tính cả chi phí debug phát sinh.

Mối quan hệ giữa năm loại handler như sau:

![Hook handler gồm command, http, mcp_tool, prompt và agent; ưu tiên dùng command script ổn định và có thể audit](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/hook-handler-types.webp)

## Hooks giải quyết vấn đề gì

Giả sử trong `CLAUDE.md` có ghi "sau khi sửa code hãy chạy Prettier". Ghi chú này sẽ đi vào context và Claude thường sẽ làm theo; nhưng khi task dài hơn hoặc có yêu cầu mới được chèn vào trong quá trình xử lý, nó cũng có thể bỏ sót. Khi rule của project chưa được sắp xếp rõ, bạn có thể xem [Best practices cho CLAUDE.md](https://javaguide.cn/ai-coding/practices/claude-md-best-practices.html).

"Không được sửa `.env`" cũng có vấn đề tương tự. Ngôn ngữ tự nhiên có thể mô tả ý định, nhưng không thể buộc kiểm tra path trước mỗi lần ghi file. Sau khi nối rule vào `PreToolUse`, script có thể đọc file đích và chặn trực tiếp khi khớp sensitive path; còn format có thể đặt trong `PostToolUse`, chỉ xử lý file vừa được sửa.

Cơ chế này tương tự pre-commit, CI, lint-staged, CODEOWNERS và branch protection: đưa các action cố định như format, kiểm tra command nguy hiểm, thông báo permission vào workflow và để lại execution result có thể kiểm tra. Hooks bổ sung chính là phần này trong lifecycle của Claude Code.

## Config Hooks tối thiểu

Hook được config trong settings file của Claude Code. Có ba vị trí thường dùng:

| Vị trí                        | Phạm vi tác dụng                               | Phù hợp để đặt gì                                      |
| ----------------------------- | ---------------------------------------------- | ------------------------------------------------------ |
| `~/.claude/settings.json`     | Tất cả project của user hiện tại               | Thông báo cá nhân, thói quen cá nhân                   |
| `.claude/settings.json`       | Project hiện tại, có thể commit vào repository | Rule dùng chung của team, giới hạn bảo mật cấp project |
| `.claude/settings.local.json` | Riêng tư trên máy local của project hiện tại   | Config cá nhân không phù hợp để commit                 |

Official còn hỗ trợ managed policy, `hooks/hooks.json` của plugin, cùng Hooks trong frontmatter của skill hoặc agent.

Khi viết project hằng ngày, chỉ cần nhớ ba vị trí trên là đủ.

Một config tối thiểu như sau:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write"
          }
        ]
      }
    ]
  }
}
```

Tách ra xem thì thực chất có ba tầng:

- `PostToolUse` là tên event, biểu thị trigger sau khi tool call thành công.
- `matcher` là điều kiện filter. Ở đây viết `Edit|Write`, chỉ trigger sau khi Claude Code dùng `Edit` hoặc `Write` để sửa file. Tool name matcher có thể phân tách bằng `|`; từ Claude Code v2.1.191, cũng có thể phân tách bằng `,` và để khoảng trắng ở hai bên. Exact matcher có hyphen cần v2.1.195 trở lên; ở version cũ có thể dùng `^...$` để giới hạn full match.
- Mảng `hooks` chứa handler thực sự được thực thi. Ở đây là một `command`, lấy path của file vừa edit từ JSON trong stdin rồi chuyển cho Prettier.

Để ví dụ ngắn, command được viết trực tiếp trong JSON. Trong project thực tế, ngay khi command bắt đầu dài hơn hoặc cần tham chiếu đến script trong project, tôi khuyên nên viết thành file độc lập rồi dùng `${CLAUDE_PROJECT_DIR}` để trỏ tới:

```json
{
  "type": "command",
  "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/format-after-edit.sh",
  "args": []
}
```

Tài liệu chính thức khuyên khi tham chiếu các placeholder như project path và plugin path nên ưu tiên exec form. Mỗi phần tử `args` sẽ được truyền cho script dưới dạng một argument độc lập, không còn đi qua shell splitting, vì vậy space, ngoặc hoặc ký tự đặc biệt trong path sẽ không bị tách lần hai.

Bỏ `matcher`, điền chuỗi rỗng hoặc dùng `.*` đều khiến hook group này match mọi lần trigger của event tương ứng. Với task format, điều này có thể khiến formatter được khởi động sau mỗi tool call; nếu đặt ở permission event, nó có thể giao mọi permission prompt cho cùng một bộ logic xử lý tự động.

Matcher nên bám sát input thực tế. Format file thì match `Edit|Write`, kiểm tra rủi ro shell thì match `Bash`, vừa giảm các call không liên quan vừa giúp xác định trong log rule nào tạo ra kết quả.

## Input/output của Hook hoạt động thế nào

Khi Hook trigger, Claude Code truyền context của event cho handler dưới dạng JSON:

- Với `command` hook, JSON này đi qua stdin.
- Với `http` hook, JSON này được gửi đến server dưới dạng POST body.

Mọi event đều có một số field dùng chung, chẳng hạn:

| Field             | Ý nghĩa                                         |
| ----------------- | ----------------------------------------------- |
| `session_id`      | ID của session hiện tại                         |
| `transcript_path` | Path đến file JSONL của session                 |
| `cwd`             | Working directory khi trigger hook              |
| `permission_mode` | Permission mode hiện tại, chỉ có ở một số event |
| `hook_event_name` | Tên event được trigger                          |

Event liên quan đến tool còn có `tool_name` và `tool_input`.

Ví dụ khi Claude Code chuẩn bị thực thi `npm test`, `PreToolUse` có thể nhận input như sau:

```json
{
  "session_id": "abc123",
  "cwd": "/Users/example/project",
  "hook_event_name": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": {
    "command": "npm test"
  }
}
```

Vì vậy, một đoạn rất thường gặp trong Hook script là:

```bash
INPUT="$(cat)"
TOOL_NAME="$(echo "$INPUT" | jq -r '.tool_name // empty')"
COMMAND="$(echo "$INPUT" | jq -r '.tool_input.command // empty')"
```

Ở đây nên dùng `jq` để parse JSON, không nên tự dùng grep để ghép field.

Ở đây đừng viết tùy tiện theo thói quen của script thông thường.

Claude Code kiểm tra stdout với mọi exit code. Nếu sau khi bỏ whitespace ở đầu, ký tự đầu tiên là `{`, nó sẽ thử parse theo JSON. Khi chuẩn bị trả về decision có cấu trúc, nên dùng `exit 0` và không đưa debug log vào stdout. Nguyên nhân lỗi và thông tin debug hãy ghi vào `stderr`. Muốn chặn thì đa số event dùng `exit 2`; lỗi non-zero thông thường nhiều khi chỉ là hook báo lỗi, workflow vẫn tiếp tục.

Điểm dễ mắc lỗi nhất là `exit 1`.

Trong shell script thông thường, `exit 1` thường biểu thị thất bại. Nhưng trong Claude Code Hooks, nếu stdout rỗng, là text thông thường hoặc JSON không vượt qua validation, `exit 1` với đa số hook event chỉ là lỗi non-blocking và workflow vẫn tiếp tục. Nếu stdout là JSON phù hợp với schema của event, các event dùng standard decision model sẽ bỏ qua exit code này và xử lý kết quả theo field trong JSON.

Tiếp theo là output JSON.

Nếu muốn điều khiển chi tiết hơn, chẳng hạn trả về `allow`, `deny`, `ask`, `defer` trong `PreToolUse`, nên dùng `exit 0`, sau đó stdout chỉ output một JSON object.

Ví dụ từ chối một tool call:

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "Database writes are not allowed"
  }
}
```

Với `PermissionRequest`, structure lại khác, trọng tâm là `decision.behavior`:

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PermissionRequest",
    "decision": {
      "behavior": "allow"
    }
  }
}
```

Đừng dùng stdout để ghi log.

Nếu muốn output JSON, stdout chỉ được chứa JSON. Thông tin debug hãy ghi vào stderr hoặc file log. Nếu không, rất dễ gặp `JSON validation failed`, rồi nhìn chằm chằm vào config mà không hiểu vấn đề.

Nếu script `exit 2`, Claude Code vẫn đọc JSON phù hợp với schema trong stdout. Với event hỗ trợ chặn, kết quả chặn của `exit 2` không thể bị JSON ghi đè; khi không có JSON chặn hợp lệ chứa nguyên nhân, Claude Code mới dùng stderr. `PermissionRequest` là ngoại lệ: nó không nhận chặn bằng `exit 2`, cả approve và deny đều phải trả về qua object `decision`.

Trong cùng một event, nếu có nhiều Hook đồng thời match, Claude Code sẽ để chúng chạy xong rồi mới merge kết quả. Một Hook trả về deny không ngăn Hook bên cạnh ghi log, gửi HTTP request hoặc sửa file; khi merge nhiều decision trong `PreToolUse`, kết quả nghiêm ngặt hơn sẽ được chọn.

Vì vậy, chỉ cần Hook có ghi log, gửi request hoặc sửa file thì nó nên tự quyết định có thực thi hay không. Đừng giả định một security Hook khác sẽ chạy trước hoặc chặn rủi ro trước.

Việc sửa tool input cũng cần thận trọng. Tài liệu chính thức đặc biệt nhắc rằng, **nếu nhiều Hook cùng cố sửa một tool input, cái hoàn thành sau cùng sẽ có hiệu lực cuối; nhưng Hook được thực thi song song nên không ổn định Hook nào hoàn thành sau cùng.**

Ngoài ra, `command` Hook trực tiếp chạy shell command với permission của user hiện tại. Nó có thể truy cập, sửa, thậm chí xóa các file mà user hiện tại có quyền thao tác, vì vậy trước khi tích hợp third-party script, nhất định phải đọc hiểu và test riêng.

![Tài liệu chính thức Claude Code cảnh báo command Hook thực thi shell command với permission của user hiện tại, có thể truy cập, sửa hoặc xóa file](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/hooks-security-warning-official-docs.png)

## Hiểu các lifecycle event thường dùng thế nào

Tài liệu chính thức liệt kê khá nhiều event, từ session, tool, permission, sub agent, task, config change, worktree cho đến MCP elicitation.

Tên event rất nhiều, nhưng lúc đầu các loại thực sự thường dùng chỉ gồm: session bắt đầu, user submit Prompt, trước khi tool thực thi, sau khi tool thực thi, quyết định permission, dừng response và nén context.

| Event               | Thời điểm trigger                                  | Phù hợp để làm gì                                               |
| ------------------- | -------------------------------------------------- | --------------------------------------------------------------- |
| `SessionStart`      | Khi session bắt đầu hoặc resume                    | Inject context động, load environment, bổ sung rule sau khi nén |
| `UserPromptSubmit`  | Sau khi user submit Prompt, trước khi Claude xử lý | Audit Prompt, chặn nhẹ, bổ sung context động                    |
| `PreToolUse`        | Trước khi tool call thực thi                       | Chặn command nguy hiểm, bảo vệ sensitive file, sửa tool input   |
| `PermissionRequest` | Khi tool call cần quyết định permission            | Audit permission hoặc auto-approve ở phạm vi rất hẹp            |
| `PostToolUse`       | Sau khi tool call thành công                       | Format, ghi log, lint, bổ sung context                          |
| `Notification`      | Khi Claude Code gửi notification                   | Notification desktop, push lên điện thoại                       |
| `Stop`              | Khi Claude hoàn thành một lượt response            | Thông báo hoàn thành, quality gate, nhắc xử lý tiếp             |
| `PreCompact`        | Trước khi nén context                              | Backup state, ngăn việc nén không phù hợp                       |
| `PostCompact`       | Sau khi nén context                                | Ghi summary, sync state bên ngoài                               |

Tiếp theo là một nhóm event nâng cao. Chỉ cần biết chúng tồn tại, khi dùng thì tra official Reference hoặc hỏi trực tiếp AI.

| Nhóm                 | Event                                                                                                      |
| -------------------- | ---------------------------------------------------------------------------------------------------------- |
| Session và config    | `Setup`, `InstructionsLoaded`, `ConfigChange`, `CwdChanged`, `DirectoryAdded`, `FileChanged`, `SessionEnd` |
| Prompt và hiển thị   | `UserPromptExpansion`, `MessageDisplay`, `TeammateIdle`                                                    |
| Tool và permission   | `PermissionDenied`, `PostToolUseFailure`, `PostToolBatch`                                                  |
| Sub agent và task    | `SubagentStart`, `SubagentStop`, `TaskCreated`, `TaskCompleted`                                            |
| Worktree và MCP form | `WorktreeCreate`, `WorktreeRemove`, `Elicitation`, `ElicitationResult`                                     |
| Bổ sung cho Stop     | `StopFailure`                                                                                              |

Một vài event dễ nhầm lẫn cần được phân biệt theo thứ tự trigger thực tế.

`PreToolUse` xảy ra trước khi tool thực thi. Kiểm tra Bash command, bảo vệ `.env` và giới hạn ghi vào config production đều nên đặt ở giai đoạn này, để script có cơ hội trả về kết quả deny trước khi side effect xuất hiện.

`PostToolUse` xảy ra sau khi tool thành công, nên phù hợp cho bước hoàn tất chứ không phù hợp làm security gate đầu tiên. Ví dụ có thể đặt format ở đây, nhưng bảo vệ sensitive file không thể chỉ dựa vào nó vì file đã bị sửa. Nó vẫn có thể dùng JSON để cung cấp feedback cho Claude hoặc thay thế tool output, chỉ là không thể undo tool call vừa xảy ra.

`PermissionRequest` trigger khi tool call cần quyết định permission và Claude Code chuẩn bị request permission, có thể approve hoặc deny request này. Trong interactive mode thường sẽ hiển thị permission confirmation dialog; background non-interactive subagent không thể hiển thị confirmation dialog vẫn chạy Hook này, và nếu không có Hook trả về decision thì tool call sẽ bị deny. Trong mode `-p` non-interactive, chỉ callback `canUseTool` của Agent SDK mới cung cấp flow permission request này; `claude -p` thông thường nên dùng `PreToolUse` để tự động quyết định permission. Risk check cần thực thi ổn định vẫn phải đặt ở `PreToolUse`; auto-approve cũng cần đồng thời thu hẹp matcher và điều kiện input để tránh allow toàn cục.

`Stop` không đồng nghĩa với "task hoàn thành", nó chỉ trigger khi Claude chuẩn bị kết thúc lượt response hiện tại. Nếu dùng Stop hook làm quality gate, cần ngăn vòng lặp. Official cung cấp field `stop_hook_active` để giúp xác định hiện tại đã được Stop hook tiếp tục hay chưa; sau khi chặn liên tiếp 8 lần, Claude Code sẽ bỏ qua việc chặn của Hook và kết thúc lượt response hiện tại.

`PreCompact` có thể ngăn việc nén, còn `PostCompact` không thể thay đổi kết quả nén đã hoàn thành. Sau khi nén, cách thường gặp để inject lại rule là dùng `SessionStart` kết hợp matcher `compact`. Nén context và bổ sung lại rule là một phần của Context Engineering; muốn tìm hiểu thêm có thể xem [Hướng dẫn thực chiến về Context Engineering](https://javaguide.cn/ai/agent/context-engineering.html).

## Ba ví dụ tối thiểu có thể dùng

Nếu thật sự bắt đầu, tôi khuyên mọi người bắt đầu từ ba ví dụ: một ví dụ chỉ phụ trách notification, một ví dụ phụ trách bước hoàn tất sau khi sửa file, và một ví dụ đặt trước khi tool thực thi để chặn.

Chúng vừa bao phủ ba scenario: rủi ro thấp, lợi ích tự động hóa và baseline bảo mật.

### Notification, bật thông báo khi Claude cần bạn

Ví dụ này phù hợp để config đầu tiên vì hầu như không đụng đến code, rủi ro thấp nhất.

Trên macOS có thể viết vào `~/.claude/settings.json`:

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "permission_prompt",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude Code needs your attention\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

Ở đây `matcher` viết `permission_prompt`, nghĩa là chỉ notification khi Claude cần bạn approve tool call. Notification này thường chỉ trigger sau khi permission request chờ khoảng 6 giây, không phải gửi ngay từ lúc flow confirmation bắt đầu. Nếu muốn trigger mọi notification, có thể bỏ matcher hoặc viết chuỗi rỗng. Các Notification matcher khác do official liệt kê còn có `idle_prompt`, `auth_success`, `elicitation_dialog` và các loại khác.

Nếu macOS không bật notification, trước tiên hãy chạy thủ công trong terminal:

```bash
osascript -e 'display notification "test"'
```

Sau đó mở quyền notification cho Script Editor trong system settings. Đây là lỗi rất thường gặp: Hook có thể đã trigger nhưng system không cho hiển thị notification.

### PostToolUse, tự động format sau khi sửa file

Trong frontend project, trường hợp thường gặp nhất là chạy Prettier sau khi sửa `Edit` hoặc `Write`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write"
          }
        ]
      }
    ]
  }
}
```

Config này có ba thông tin quan trọng.

`matcher` chỉ match `Edit|Write`, vì vậy đọc file, chạy Bash hoặc gọi MCP tool đều không trigger format.

`command` lấy `.tool_input.file_path` từ stdin JSON rồi chuyển cho `npx prettier --write`.

Hook này nằm trên `PostToolUse`, nên nó là bước "hoàn tất sau khi tool thực thi". Khi formatter thất bại, bạn có thể để lỗi lộ ra hoặc đổi thành script, chọn formatter khác nhau theo file extension.

Ví dụ script ổn định hơn:

```bash
#!/usr/bin/env bash
set -euo pipefail

file="$(jq -r '.tool_input.file_path // empty')"

case "$file" in
  *.js|*.jsx|*.ts|*.tsx|*.json|*.md)
    npx prettier --write "$file"
    ;;
esac
```

Hook không có phép màu. Nếu là Java project, nên đổi sang `spotlessApply`, `google-java-format` hoặc format command đã có trong project. Nếu là Python project, có thể dùng `ruff format`. Trước tiên hãy bám theo tool hiện có của project, đừng tạo mới cả một hệ thống format chỉ để viết Hook.

### PreToolUse, chặn command nguy hiểm và sensitive file

`PreToolUse` cung cấp `tool_name` và `tool_input` trước khi tool thực thi. Script có thể kiểm tra command và file path trước, khi match risk thì trực tiếp deny call này.

Trước tiên viết một script, chẳng hạn `.claude/hooks/guard.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

input="$(cat)"
tool="$(jq -r '.tool_name // empty' <<<"$input")"
command="$(jq -r '.tool_input.command // empty' <<<"$input")"
file="$(jq -r '.tool_input.file_path // empty' <<<"$input")"

if [[ "$tool" == "Bash" ]] && [[ "$command" =~ rm[[:space:]]+-rf|chmod[[:space:]]+-R[[:space:]]+777 ]]; then
  echo "Blocked risky shell command: $command" >&2
  exit 2
fi

if [[ "$tool" == "Edit" || "$tool" == "Write" ]]; then
  case "$file" in
    *.env|*.env.*|*/.env|*/.git/*|*id_rsa*|*id_ed25519*)
      echo "Blocked sensitive file edit: $file" >&2
      exit 2
      ;;
  esac
fi

exit 0
```

Trao quyền thực thi cho nó:

```bash
chmod +x .claude/hooks/guard.sh
```

Sau đó gắn vào `.claude/settings.json` cấp project:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash|Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/guard.sh"
          }
        ]
      }
    ]
  }
}
```

Script này được đặt trước khi tool thực thi và biểu đạt kết quả xử lý qua return value. Sau khi match risk, nó ghi nguyên nhân vào stderr rồi thực hiện `exit 2`; Claude Code sẽ chặn tool call này, đồng thời feedback nguyên nhân cho Claude.

Trong project thực tế, sensitive list cần được sửa theo tình hình của bạn. Config production, credential file, migration script, lock file và CI config đều có thể lần lượt thêm vào.

Ở đây đừng chỉ dựa vào một command blacklist để làm hàng rào cuối. Ví dụ chỉ chặn `rm *` không có nghĩa là chặn được các biến thể như `/bin/rm` và `find -delete`. Thao tác rủi ro cao tốt nhất nên kết hợp path restriction, permission config, Hooks, Sandbox, CI và manual Review.

## Chọn non-command Hook thế nào

`command` phù hợp với task xác định có thể hoàn thành trên máy local; format, notification và command interception ở trên đều thuộc loại này. Script có thể debug độc lập và cũng dễ đưa vào code review.

Khi team cần tập trung thu thập audit record hoặc thực thi policy từ xa, có thể dùng `http`. JSON body server trả về sẽ được xử lý theo format JSON output của command hook. Bản thân HTTP status code không phụ trách chặn tool call; server cần trả về 2xx và cung cấp field decision phù hợp với schema trong response body.

`mcp_tool` dùng để gọi MCP server đã kết nối. Nó không trigger OAuth và cũng không tự thiết lập connection thay bạn. Các event như `SessionStart`, `Setup` xảy ra khá sớm; nếu MCP server chưa sẵn sàng vào lúc đó, call sẽ fail trực tiếp.

`prompt` thực hiện một lần phán đoán của model dựa trên Hook input, ví dụ kiểm tra task đã hoàn thành trước Stop. Nếu xác minh còn cần đọc file, tìm code hoặc chạy command thì có thể dùng `agent`, nhưng hiện tại đây vẫn là tính năng experimental.

Khi lựa chọn, trước tiên hãy xem căn cứ phán đoán nằm ở đâu: rule hoàn toàn do input field và script quyết định thì dùng `command`; kết quả cần đi vào service của team thì dùng `http`; đã có MCP tool dùng được thì chọn `mcp_tool`; chỉ khi phán đoán ngữ nghĩa không thể viết thành rule xác định mới đưa `prompt` hoặc `agent` vào.

## Phân biệt Hooks và Skills thế nào

Hai khái niệm này cũng đặc biệt dễ nhầm.

Skills mở rộng năng lực của Coding Agent thông qua `SKILL.md`. Khi thực thi task, Coding Agent sẽ chủ động quyết định có dùng skill liên quan hay không; bạn cũng có thể gọi tường minh bằng `/skill-name`.

Nội dung Skill chỉ được load vào context khi sử dụng (progressive loading), vì vậy rất phù hợp để tích lũy workflow dài, checklist, kiến thức project, script và tài liệu tham khảo.

![Progressive disclosure của Skill](https://oss.javaguide.cn/github/javaguide/ai/skills/agent-skills-progressive-disclosure.webp)

Nếu muốn hiểu có hệ thống sự phân công giữa Skills với Prompt, MCP và Function Calling, có thể xem [Agent Skills là gì? Khác Prompt và MCP ở đâu?](https://javaguide.cn/ai/agent/skills.html).

![Execution flow của Agent](https://oss.javaguide.cn/github/javaguide/ai/skills/skill-agent-execution-link.webp)

Hooks tự động thực thi action tại lifecycle node, còn Skills đưa cho Claude các instruction, script và tài liệu tham khảo cần để hoàn thành một loại task. Có thể phân biệt hai loại này theo bảng sau:

![Hooks phù hợp với trigger tự động, action cố định và chặn bảo mật; Skills phù hợp với việc load theo nhu cầu, phán đoán theo context và workflow phức tạp](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/hooks-vs-skills-responsibilities.webp)

| Dimension                             | Hooks                                                                             | Skills                                                                                    |
| ------------------------------------- | --------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| Cách trigger                          | Tự động trigger theo lifecycle event của Claude Code                              | Claude load khi phán đoán có liên quan hoặc user gọi thủ công `/skill-name`               |
| Giá trị cốt lõi                       | Đảm bảo action cố định luôn xảy ra                                                | Bổ sung cho Claude năng lực hoặc kiến thức workflow của một loại task                     |
| Trường hợp phù hợp                    | Format, chặn command nguy hiểm, audit permission, notification, log, quality gate | Quy trình code review, deploy SOP, troubleshooting, tra cứu tài liệu, xử lý task phức tạp |
| Mức phụ thuộc vào phán đoán của model | Thấp, đặc biệt với `command` hook                                                 | Cao hơn, Claude cần hiểu và thực thi instruction của skill                                |
| Có phù hợp để chặn không              | Phù hợp, đặc biệt là các event `PreToolUse`, `UserPromptSubmit`, `Stop`           | Không phù hợp làm cơ chế hard block                                                       |
| Risk thường gặp                       | matcher quá rộng, script chậm, auto-approve quá mức                               | Mô tả không rõ, trigger không chính xác, workflow quá dài                                 |

Đặt vào task thực tế sẽ dễ hiểu hơn. Ngay sau khi file bị `Edit` hoặc `Write` sửa, có thể trực tiếp chuyển path cho Prettier; khi tool chuẩn bị ghi vào `.env`, cũng có thể kiểm tra path trước khi thực thi và deny. Các thông tin này đều lấy được từ event input, script không cần đoán.

Code review và troubleshooting timeout của API thì không cố định như vậy. Claude phải đọc code, log và yêu cầu task trước, sau đó mới quyết định cần kiểm tra file nào và chạy command nào; loại workflow này phù hợp hơn khi đặt trong Skill.

Hai loại cũng có thể nối vào cùng một workflow: **Skill quy định code review cần kiểm tra những gì, Hook phụ trách chạy formatter sau khi sửa file, chặn trước khi command nguy hiểm thực thi, và kiểm tra trước khi response kết thúc xem có để lại test result hay không.**

## Triển khai thực tế: trước tiên làm ổn định ba Hook

Ở phiên bản đầu không cần vội bao phủ mọi lifecycle event, lần lượt tích hợp `Notification`, `PostToolUse`, `PreToolUse` là đủ.

![Claude Code Hooks được khuyên tích hợp từng bước theo thứ tự Notification, PostToolUse, PreToolUse](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/hooks-progressive-rollout.webp)

Càng config nhiều Hook, khi có vấn đề càng khó xác định. Ví dụ Claude đột nhiên không response nữa, bạn phải lần lượt xác nhận: `PreToolUse` có trả về deny không, `PermissionRequest` có đưa ra permission decision không, `Stop` có trigger lặp lại không, có phải một `PostToolUse` script nào đó chạy quá timeout không. Bề ngoài các vấn đề này khá giống nhau, nhưng cách troubleshooting lại hoàn toàn khác.

Trước tiên tích hợp `Notification`. Ví dụ ở trên chỉ gửi notification khi chờ authorization, không sửa file project. Nhận được notification ít nhất có thể xác nhận config đã được load, event và matcher cũng match thành công. Kể cả viết sai, ảnh hưởng thường cũng nhỏ.

Sau khi notification hoạt động bình thường, nối `PostToolUse` vào format tool đã có của project. Frontend project dùng Prettier, Python project dùng Ruff, Java project tiếp tục dùng format command vốn có. Matcher chịu trách nhiệm giới hạn tool name, script tiếp tục filter theo file type; đừng để một Bash command không liên quan hoặc thao tác đọc cũng khởi động formatter.

Cuối cùng mới test `PreToolUse`, vì nó sẽ trực tiếp chặn tool call. Ban đầu chỉ chặn một vài thao tác rủi ro cao rõ ràng, chẳng hạn command xóa, nâng quyền đệ quy và truy cập `.env`, `.git/`, private key hoặc config production. Trước tiên quan sát kết quả chặn từ log, sau đó điều chỉnh rule dựa trên các trường hợp chặn nhầm.

Blacklist command chỉ nhận diện được dạng command đã viết ra, đổi cách viết có thể vượt qua. Path restriction, permission config, sandbox và manual Review vẫn phải được giữ lại, không được vì thêm `PreToolUse` mà xóa chúng.

Sau khi ba Hook này chạy ổn định, hãy bổ sung event theo các vấn đề thực tế trong project. Nếu thường xuyên mất rule sau khi nén context, có thể dùng matcher `compact` của `SessionStart` để inject lại, khi cần thì dùng `PreCompact` lưu state hiện tại. Khi cần theo dõi config change thì nối `ConfigChange`; khi thay đổi working directory hoặc file ảnh hưởng đến environment, mới cân nhắc dùng `CwdChanged`, `FileChanged` kết hợp với `CLAUDE_ENV_FILE`.

Nếu dùng `Stop` để kiểm tra hoàn thành, script nhất định phải viết cả điều kiện thoát. Nếu không, sau khi Hook khiến Claude tiếp tục xử lý, khi lượt tiếp theo kết thúc lại match cùng rule và lặp đi lặp lại. Claude Code sẽ force kết thúc lượt hiện tại sau 8 lần chặn liên tiếp, nhưng việc xử lý lặp ở phía trước vẫn lãng phí thời gian và token.

`PermissionRequest` cũng rất dễ dùng sai. Nó trigger khi tool call đi vào flow quyết định permission, không có nghĩa confirmation dialog đã hiển thị; background non-interactive subagent không thể hiển thị confirmation dialog cũng có thể chạy Hook này.

`claude -p` thông thường không có flow permission request này. Khi cần tự động quyết định permission, nên đặt rule vào `PreToolUse`. Ví dụ chính thức chỉ auto-approve `ExitPlanMode`. Khi sao chép ví dụ, cần giữ cả matcher và event input, đừng dùng một allow rule quá rộng để tiếp quản mọi permission request. Xóa file, thao tác trên production environment, đọc credential hoặc gọi external API để ghi data vẫn giao cho người xác nhận sẽ an toàn hơn.

## Hook không hoạt động như mong đợi thì troubleshooting thế nào

Config đã viết vào nhưng Claude Code hoàn toàn không phản ứng; hoặc Hook rõ ràng đã chạy mà command nguy hiểm vẫn tiếp tục thực thi. Các chi tiết như event có field nào, một `handler` hỗ trợ event nào, stdout có đi vào context hay không đều liên quan đến version, khi cần chỉ cần để AI đối chiếu tài liệu chính thức hiện tại.

Khi thật sự gặp vấn đề, tôi khuyên nên troubleshooting theo thứ tự dưới đây:

![Khi Claude Code Hook không có hiệu lực, troubleshooting từng lớp từ load config, chạy script độc lập đến matcher](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/hooks-troubleshooting-flow.webp)

1. Trước tiên chạy `/hooks`, xác nhận config đã được load và được gắn vào event mong muốn. Nếu không thấy ở đây, trước tiên kiểm tra vị trí settings file và format JSON, tạm thời chưa cần quan tâm logic script.

   ![Tài liệu chính thức Claude Code giải thích menu /hooks có thể xem chi tiết event, matcher, handler và nguồn config](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/hooks-menu-official-docs.png)

2. Lấy script ra khỏi Claude Code và chạy riêng. Hook script đọc JSON từ `stdin`, trước tiên xác nhận nó đọc được field và trả về exit code như dự kiến, sau đó mới gắn lại vào config.

   ```bash
   printf '%s\n' '{"tool_name":"Bash","tool_input":{"command":"rm -rf /tmp/demo"}}' \
     | .claude/hooks/guard.sh

   echo $?
   # Đầu ra dự kiến: 2
   ```

3. Sau khi script chạy riêng bình thường, hãy ghi lại event data thực tế Claude Code truyền vào. Thông tin debug ghi vào `stderr` hoặc log tạm, đừng trộn vào `stdout` đang chuẩn bị trả về JSON.

   ```bash
   input="$(cat)"
   printf '%s\n' "$input" >> /tmp/claude-hook-debug.log
   ```

   Log có thể chứa file path, command và nội dung Prompt; sau khi troubleshooting xong hãy nhớ xóa, không commit vào repository.

4. Mỗi lần chỉ enable một Hook. Khi nhiều Hook đồng thời match, chúng chạy song song nên chỉ nhìn biểu hiện cuối rất khó xác định rule nào có vấn đề. Sau khi một script chạy thông, lần lượt khôi phục các rule khác.

Nếu đã xuất hiện symptom rõ ràng, có thể kiểm tra trước mục tương ứng:

| Hiện tượng                                           | Ưu tiên kiểm tra                                                                         |
| ---------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Không thấy config trong `/hooks`                     | Vị trí settings file, format JSON                                                        |
| Đã register nhưng không bao giờ trigger              | Event có chọn đúng không, matcher có match input thực tế không                           |
| Script báo lỗi nhưng tool call vẫn tiếp tục          | Có dùng nhầm `exit 1` không; security interception có đặt ở `PreToolUse` không           |
| Claude không ngừng xử lý, không thể kết thúc         | `Stop` Hook có thiếu exit condition không                                                |
| Mỗi thao tác đều chạy formatter                      | matcher có bị bỏ qua hoặc viết quá rộng không                                            |
| Notification script đã chạy nhưng không có popup     | Chạy thủ công notification command trước, sau đó kiểm tra system notification permission |
| Interactive mode bình thường, `claude -p` bất thường | Đừng phụ thuộc vào `PermissionRequest`; khi cần auto decision thì đổi sang `PreToolUse`  |

Nếu vẫn chưa xác định được, có thể đưa output của `claude --version`, Hook config, script và hiện tượng thực tế cho AI, để nó đối chiếu tài liệu chính thức hiện tại. Trước khi paste, nhớ xóa secret, Token và thông tin nhạy cảm của business. Prompt không cần viết phức tạp:

```text
Tôi đã config Hook dưới đây trong Claude Code <version>.

Hành vi mong muốn: <muốn nó làm gì>
Hành vi thực tế: <hiện tại đang xảy ra gì>

Hook config:
<paste config>

Script:
<paste script>

Trước tiên hãy tra tài liệu chính thức của version Claude Code hiện tại, sau đó lần lượt kiểm tra event, matcher, input field, exit code và format output JSON.
Chỉ chỉ ra những điểm gây ra vấn đề hiện tại, không mở rộng sang config khác.
```

## Tổng kết

Đến đây, bạn hẳn đã nắm được trọng tâm của Hooks: chúng tự động thực thi action cố định tại các node quan trọng của Claude Code, chẳng hạn format, kiểm tra command nguy hiểm và thông báo permission.

Đặt auto format ở `PostToolUse`, đặt chặn command nguy hiểm và sensitive file ở `PreToolUse`, còn khi rời máy mà muốn kịp thời nhận nhắc nhở thì dùng `Notification`.

Theo kinh nghiệm sử dụng cá nhân của tôi, không nên config quá nhiều ngay từ đầu, không có nhiều ý nghĩa mà còn dễ rối.

Trước tiên làm ổn định ba Hook này. Đặc biệt với rule security và permission, thà thu hẹp phạm vi rồi bổ sung dần còn hơn auto-approve mọi thao tác ngay từ đầu. Hooks có thể giúp bạn giữ workflow cố định, nhưng sandbox, permission config, CI và manual Review vẫn cần được giữ lại.

Một điểm khác cũng dễ nhầm: rule có thể viết thành script xác định thì giao cho Hook; task như code review và troubleshooting cần kết hợp phán đoán theo context thì tiếp tục đặt trong Skill. Phân công theo cách này sẽ giúp config đơn giản hơn nhiều và troubleshooting về sau cũng nhẹ nhàng hơn.
