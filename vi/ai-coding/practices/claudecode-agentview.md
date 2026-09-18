---
title: "Claude Code Agent View: Thực chiến quản lý nhiều Session song song"
description: Anthropic phát hành Agent View, cung cấp cho Claude Code khả năng quản lý nhiều Session trong terminal, tập trung xem trạng thái Agent, xử lý input và quản lý Session chạy nền.
category: AI Coding thực chiến
head:
  - - meta
    - name: keywords
      content: Claude Code,Agent View,quản lý nhiều Session,Agent song song,AI Coding,công cụ CLI,điều phối Session
---

Xin chào, tôi là Tiểu G.

Tôi thường dùng Claude Code và thường mở nhiều Session cùng lúc: một Session phát triển tính năng mới, một Session refactor, một Session chạy test, một Session xem lỗi, Session khác tổng hợp bình luận PR hoặc bổ sung tài liệu.

![Mở nhiều cửa sổ dòng lệnh để nhiều Agent chạy song song trong các Session khác nhau](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/multi-agent-parallel-sessions.png)

Trước đây dùng như vậy thực sự khá mệt. Tôi thường mở nhiều split trong Ghostty, rồi thêm vài tab terminal. Các cửa sổ phủ kín màn hình, trông như đã tối đa hóa hiệu suất song song, nhưng trong đầu vẫn phải nhớ: Session nào còn đang chạy? Session nào đã hoàn tất? Session nào đang mắc ở bước xác nhận quyền? Session nào bị lỗi?

Phiền nhất là có những Agent thực ra đã chờ bạn xác nhận từ lâu, nhưng bạn hoàn toàn không để ý. Đến khi quay lại, nó đã dừng ở đó hàng chục phút.

**Agent View** do Anthropic phát hành cách đây không lâu vừa khéo tiếp nhận việc phiền phức này. Nó tập trung các Session chạy nền vào một danh sách: đang làm việc, chờ input, đã hoàn tất hay chạy thất bại, chỉ cần liếc qua là biết. Claude vẫn là Claude đó, nhưng cuối cùng tôi không còn phải dùng trí nhớ để duy trì “bảng trạng thái Session” nữa.

Trong bài [Chọn CLI hay IDE khi lập trình AI?](https://mp.weixin.qq.com/s/6a3f2U6ZAJa2N7Cp10S01Q), tôi từng đề cập rằng một số workflow mới của AI Coding thường được thử nghiệm trước trong CLI. Agent View là một ví dụ. Tuy nhiên, nó gần với một “trình quản lý Session chạy nền” hơn, chưa phải nền tảng điều phối multi-Agent có thể tự động chia task, phân công và điều phối conflict.

Nếu bạn chưa quen với Claude Code, có thể xem trước hai bài sau:

- [《Hướng dẫn sử dụng Claude Code》](https://javaguide.cn/ai-coding/practices/claudecode-tips.html): Sub-Agent, phối hợp nhiều instance (Multi-Claude), cấu hình CLAUDE.md, v.v.
- [《Giải thích chi tiết các command cốt lõi của Claude Code》](https://javaguide.cn/ai-coding/practices/claudecode-commands.html): cách dùng thực tế các command như `/simplify`, `/loop`, `/batch`, v.v.

## Mở Agent View như thế nào

Agent View hiện vẫn là Research Preview (bản preview nghiên cứu), yêu cầu Claude Code `v2.1.139` hoặc cao hơn. Trước tiên có thể kiểm tra version:

```bash
claude --version
```

Cách mở trực tiếp nhất là chạy trong terminal:

```bash
claude agents
```

![Chạy trực tiếp claude agents trong terminal để truy cập](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/claude-agents-list-view.png)

Sau khi mở, mỗi Session chạy nền chiếm một dòng. Bên trái là icon trạng thái, ở giữa là tên Session và tóm tắt thực thi gần nhất, bên phải là thời gian chạy. Session mặc định được nhóm theo trạng thái; những Session cần bạn xử lý sẽ được xếp lên trước.

Các Session Claude Code thông thường đã mở sẽ không tự động xuất hiện ở đây. Muốn chuyển Session hiện tại vào nền, có thể nhập:

```text
/bg
```

Cũng có thể nhấn phím mũi tên trái `←` khi ô input đang trống. Cả hai thao tác đều tách Session vào nền, không kết thúc task. Sau đó dùng phím mũi tên chọn Session, rồi nhấn `Enter` hoặc `→` để vào lại cuộc hội thoại đầy đủ.

![Vào Session Agent được chỉ định](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/enter-agent-session.png)

## Xem màu vàng trước, màu đỏ sau

Sau khi mở Agent View, tôi thường quét qua icon trạng thái bên trái trước. Nó đáng xem hơn tên Session vì cho biết trực tiếp nơi nào cần can thiệp.

![Claude Code Agent View](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/claude-agents-list-view-20260518102539932.png)

| Trạng thái    | Hiển thị trên giao diện | Cách xử lý                                                                   |
| ------------- | ----------------------- | ---------------------------------------------------------------------------- |
| `Needs Input` | màu vàng                | Đang chờ câu trả lời, xác nhận quyền hoặc đăng nhập, ưu tiên xử lý           |
| `Working`     | animation               | Vẫn đang gọi tool hoặc tạo câu trả lời, có thể để đó trước                   |
| `Completed`   | màu xanh                | Task kết thúc bình thường, tiếp theo kiểm tra Diff, test và kết quả thực thi |
| `Failed`      | màu đỏ                  | Chạy lỗi, mở tóm tắt hoặc log để xác định nguyên nhân                        |
| `Idle`        | mờ đi                   | Hiện không có task, có thể tiếp tục gửi message                              |
| `Stopped`     | màu xám                 | Session đã bị dừng thủ công hoặc process bị kết thúc từ bên ngoài            |

Có một điểm dễ hiểu nhầm: nhóm `Completed` trên giao diện cũng chứa các Session thất bại và đã dừng, nên tên nhóm không có nghĩa tất cả task đều thành công. Muốn biết có thực sự hoàn tất hay không, vẫn phải xem màu icon và kết quả thực thi.

Màu vàng hữu ích nhất. Trước đây tôi tưởng một Session nào đó vẫn đang chạy, đến khi quay lại mới phát hiện nó đã dừng từ lâu ở câu “Có cho phép thực thi command này không?” để chờ tôi. Bây giờ thấy màu vàng thì xử lý, không có màu vàng thì làm việc khác trước.

## Không cần chuyển đổi cũng có thể trả lời

Sau khi chọn Session, nhấn phím cách `Space`, bên dưới sẽ hiện Peek Panel, hiển thị output gần nhất hoặc câu hỏi Claude đang chờ.

![Nhấn phím cách sau khi chọn một Session trong Agent View để mở Peek Panel](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/peek-panel-reply.png)

Nếu chỉ cần xác nhận “Có cho phép sửa file này không?” hoặc “Có tiếp tục chạy test không?”, bạn có thể trả lời trực tiếp trong panel. Sau khi nhận message, Session sẽ tiếp tục thực thi mà không cần vào cuộc hội thoại đầy đủ.

Trước đây phải tìm tab terminal tương ứng, xem nó đang chờ gì, trả lời xong lại quay về. Bây giờ chỉ cần nhấn phím cách một lần là xử lý được; những chi tiết nhỏ như vậy dùng lâu sẽ rất nhẹ đầu.

Nếu muốn xem context đầy đủ, nhấn `Enter` hoặc `→` để vào Session; xem xong nhấn `←` để quay lại Agent View.

Không cần học thuộc shortcut riêng; gợi ý sẽ hiển thị ở cuối giao diện, nhấn `?` còn có thể xem danh sách đầy đủ. Các shortcut thường dùng hằng ngày chủ yếu là:

| Shortcut          | Chức năng                                             |
| ----------------- | ----------------------------------------------------- |
| `↑` / `↓`         | Di chuyển trong danh sách Session                     |
| `Space`           | Mở hoặc đóng Peek Panel                               |
| `Enter`           | Vào Session, hoặc dispatch task khi input có nội dung |
| `Shift+Enter`     | Dispatch task và lập tức vào Session mới              |
| `Alt+1` ~ `Alt+9` | Vào trực tiếp Session thứ N                           |
| `Ctrl+T`          | Ghim hoặc bỏ ghim Session hiện tại                    |
| `Ctrl+R`          | Đổi tên Session hiện tại                              |
| `Ctrl+X`          | Dừng Session; nhấn lại trong 2 giây để xóa            |

Cần thận trọng khi nhấn `Ctrl+X` hai lần liên tiếp. Nếu Session sử dụng Worktree do Claude Code tự tạo, các thay đổi chưa commit bên trong cũng có thể bị xóa khi xóa Session.

## Đẩy task chạy nền

Nhập `/bg` trong Session hiện có để trực tiếp đưa task hiện tại vào nền:

```text
/bg
```

Thao tác này đưa Session hiện tại vào nền, sau đó quay lại Agent View.

![Đưa task chạy nền bằng /bg](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/bg-background-session.png)

Cũng có thể tiện thể thêm một instruction rồi chuyển vào nền:

```text
/bg chạy test và sửa các case thất bại
```

Nếu task chưa bắt đầu, tạo trực tiếp một Session chạy nền từ Shell sẽ tiện hơn:

```bash
claude --bg "Sửa tất cả unit test thất bại trong module auth cho đến khi tất cả đều pass"
```

Những task “tốn thời gian nhưng không cần theo dõi liên tục” rất phù hợp để chạy nền:

- Chạy cả nhóm test thất bại và thử sửa
- Kiểm tra lỗi type của một module
- Tổng hợp tài liệu hàng loạt
- Phân tích bình luận PR và đưa ra đề xuất sửa đổi
- Thực hiện các thay đổi nhỏ trên nhiều repository cùng lúc

Mitchell Hashimoto (đồng sáng lập HashiCorp, tác giả Ghostty) từng chia sẻ một thói quen khá thú vị trong [My AI Adoption Journey](https://mitchellh.com/writing/my-ai-adoption-journey): 30 phút cuối mỗi ngày, giao việc nghiên cứu chuyên sâu, khám phá các ý tưởng chưa rõ ràng, phân loại Issue và PR cho Agent, sáng hôm sau đi làm chỉ cần xem kết quả.

Ông cũng viết khá chừng mực: trạng thái lý tưởng là luôn có Agent xử lý công việc hữu ích, nhưng thực tế chỉ đạt được trong khoảng 10%～20% thời gian làm việc, và thường chỉ chạy một Agent. Tỷ lệ này ngược lại gần với việc phát triển hằng ngày hơn, không cần cố ép một hàng task chỉ để “chạy song song”.

## Shell command

Nếu không muốn mở Agent View, bạn cũng có thể quản lý Session chạy nền trực tiếp trong Shell:

```bash
claude agents          # mở Agent View

claude attach <id>     # chuyển đến Session được chỉ định

claude logs <id>       # in output gần nhất của Session được chỉ định

claude stop <id>       # dừng Session, cũng có thể dùng claude kill

claude respawn <id>    # khởi động lại Session, giữ lại lịch sử hội thoại

claude respawn --all   # khởi động lại tất cả Session chạy nền

claude rm <id>         # xóa Session khỏi danh sách
```

Trong số này, command tôi ít dùng hơn nhưng rất dễ hiểu nhầm là `respawn`:

```bash
claude respawn <id>
```

Nó khởi động lại Session được chỉ định và giữ nguyên lịch sử hội thoại. Sau khi Claude Code cập nhật và muốn một Session chạy nền nào đó dùng version mới, hoặc khi process của Session thoát bất thường, đều có thể dùng command này.

`respawn` không phải Context Reset và cũng không tạo context sạch. Nếu Session cũ đã chứa đầy log không liên quan và phương án hết hiệu lực, khởi động lại process cũng không giải quyết được context pollution. Trong trường hợp này, mở Session khác và đưa vào một bản tóm tắt task đã kiểm tra sẽ an toàn hơn.

Ngoài ra, `claude rm <id>` chủ yếu xóa Session khỏi danh sách chạy nền. Lịch sử hội thoại vẫn được lưu trên máy local, có thể dùng `claude --resume` để tìm lại; Worktree do Claude Code tự tạo chỉ được dọn cùng khi xác nhận an toàn.

## Dispatch task trực tiếp từ Agent View

Bên dưới Agent View có một ô input. Nhập task rồi nhấn `Enter` sẽ tạo một Session chạy nền mới; nhập task tiếp theo sẽ tiếp tục tạo Session mới thay vì hỏi thêm Session trước đó. Muốn bổ sung thông tin cho Session hiện có, cần trả lời bằng Peek Panel hoặc vào Session trước.

Ô input cũng hỗ trợ một số cách viết đặc biệt:

| Định dạng input         | Hiệu quả                                                             |
| ----------------------- | -------------------------------------------------------------------- |
| `<agent-name> <prompt>` | Khi từ đầu tiên khớp với Subagent tùy chỉnh, dùng nó làm Agent chính |
| `@<agent-name>`         | Chỉ định Subagent tùy chỉnh trong Prompt                             |
| `@<repo>`               | Chọn repository hoặc directory khác để khởi động Session tại đó      |
| `/<command>`            | Tìm và dùng Skill hoặc command có thể dispatch                       |
| `! <command>`           | Trực tiếp khởi động task Shell chạy nền, không gọi model             |
| `#<number>` hoặc PR URL | Khi Session hiện có đang xử lý PR đó, trực tiếp chọn Session gốc     |

Subagent tùy chỉnh thường được đặt trong `.claude/agents/` của project hoặc `~/.claude/agents/` của user. Chuẩn bị một Agent chuyên dụng cho các task lặp lại như code review, phân tích test; khi dispatch sẽ không cần nhắc lại tool và phạm vi mỗi lần.

`#<number>` và PR URL cũng khá hữu ích. Nếu đã có một Session xử lý cùng PR, Agent View sẽ chọn Session gốc, tránh hai Session sửa trùng nhau.

Về cách dùng Skills, có thể xem tiếp [6 Skills được đề xuất](https://mp.weixin.qq.com/s/55YhKrMAHsbrAgf4P2ezRA) và [Giải thích chi tiết Agent Skills](https://mp.weixin.qq.com/s/5iaTBH12VTH55jYwo4wmwA).

## Những trường hợp phù hợp để sử dụng

Khi quyết định có nên đưa task vào Agent View hay không, tôi chủ yếu xem hai điểm: nó có thể tạm thời tự tiến hành độc lập khi tôi rời đi không, và khi quay lại có thể nghiệm thu bằng Diff, test hoặc log không. Đáp ứng cả hai điểm thì chạy nền thường khá nhẹ đầu.

### Xử lý đồng thời nhiều việc nhỏ không phụ thuộc nhau

Ví dụ có 5 yêu cầu nhỏ, có thể giao cho 5 Session: một Session sửa test, một Session tìm lỗi type, một Session tổng hợp tài liệu, hai Session còn lại xử lý các Bug không liên quan nhau. Sau khi dispatch task, hãy làm việc riêng trước, quay lại xem trạng thái sau: trả lời màu vàng trước, xem log của màu đỏ, nghiệm thu tập trung các Session màu xanh.

Điều kiện tiên quyết là các task này thực sự có thể tách rời. Năm Session cùng sửa một nhóm file, dù có Worktree isolation, cuối cùng vẫn phải xử lý conflict và thay đổi trùng lặp. Thời gian tiết kiệm được rất có thể lại mất vào việc merge code.

[Thí nghiệm compiler C của Nicholas Carlini](https://www.anthropic.com/engineering/building-c-compiler) đã nâng quy mô chạy song song lên một cấp độ khác: 16 instance Claude Opus 4.6 đã chạy gần 2000 Claude Code Session trong hai tuần, tạo ra khoảng 100 nghìn dòng code, chi phí gần 20 nghìn USD. Thí nghiệm này sử dụng Agent Teams và execution framework tùy chỉnh, không phải minh họa khả năng của Agent View.

Trong phát triển hằng ngày, điều có thể học hỏi là chia task, phân công role và ràng buộc bằng test. Thiếu các bước chuẩn bị này, mở thêm vài Session chỉ khiến conflict phát sinh nhanh hơn. Cách dùng cụ thể của các workflow song song như `/simplify` và `/batch` có thể xem trong [《Giải thích chi tiết các command cốt lõi của Claude Code》](https://javaguide.cn/ai-coding/practices/claudecode-commands.html).

### CI, test và PR cần thời gian chờ

CI, integration test và PR Review thường phải chờ kết quả bên ngoài. Để Session ở foreground liên tục khiến người ta cứ vài phút lại muốn xem một lần; đưa vào nền, chờ trạng thái chuyển vàng, đỏ hoặc hoàn tất rồi quay lại xử lý là được.

Có thể dùng `/loop` để kiểm tra CI định kỳ; với task có điều kiện kết thúc rõ ràng, chẳng hạn “liên tục sửa cho đến khi tất cả test pass”, dùng `/goal` phù hợp hơn. Agent View chỉ phụ trách hiển thị trạng thái Session; việc kiểm tra theo lịch và thực thi liên tục vẫn do các command này hoàn thành.

[Minions của Stripe](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents) đã biến chuỗi này thành một hệ thống nội bộ: developer dispatch task từ Slack, Agent phụ trách coding, verification và tạo PR; mỗi tuần có hơn 1000 PR kiểu này được merge, code vẫn do người review. Minions và Agent View là hai hệ thống khác nhau, nhưng đều không thể thiếu test, CI và Code Review. Thiếu các bước này, danh sách hiển thị màu xanh cũng không chứng minh code đã có thể merge.

### Chạy song song trên nhiều repository

`@<repo>` có thể dispatch task đến repository hoặc directory khác. Một Session sửa API backend, một Session sửa frontend, một Session khác cập nhật tài liệu; trạng thái vẫn nằm trong cùng một danh sách, không cần quản lý riêng nhiều nhóm cửa sổ terminal.

Trong Git repository, lần đầu Session chạy nền chuẩn bị sửa file, Claude Code mặc định chuyển nó vào Worktree độc lập dưới `.claude/worktrees/`, tránh việc nhiều Session ghi trực tiếp vào cùng một workspace. Với directory không phải Git, đang ở trong Worktree, hoặc project đã tắt Worktree isolation cho Session chạy nền, hành vi sẽ khác.

Nếu task frontend và backend phụ thuộc cùng một API contract, tốt nhất hãy chốt trước request parameter, response structure và tiêu chuẩn nghiệm thu. Agent View không đồng bộ các thay đổi này thay cho hai Session.

## Những trường hợp không nên cố dùng

### Nhiều task cần đồng bộ thường xuyên

Mỗi Session có context riêng và chỉ báo cáo tiến độ cho bạn. Một Session vừa sửa xong API thì Session khác không tự biết; hai Session cùng điều chỉnh một module cốt lõi cũng không chủ động bàn bạc ai phụ trách.

Các task phụ như tra cứu tạm thời, chạy test có thể giao cho Subagent; nếu cần nhiều Agent chia sẻ task list và trao đổi với nhau, có thể cân nhắc Agent Teams. Agent View phù hợp hơn với việc con người phân công các task độc lập rồi thống nhất kiểm tra kết quả.

Các Session chạy song song cũng lần lượt tiêu hao hạn mức subscription. Càng mở nhiều task, Token tiêu thụ và tốc độ chạm giới hạn càng nhanh. Nếu vài việc vốn phải chờ lẫn nhau, xử lý tuần tự có thể tiện hơn.

### Task bắt buộc chạy liên tục khi máy local đã ngắt kết nối

Session chạy nền do process Supervisor trên máy local quản lý. Sau khi đóng Agent View, Shell hoặc cửa sổ terminal, task vẫn có thể tiếp tục; khi máy sleep, Session sẽ được giữ lại và kết nối lại sau khi đánh thức.

Tắt máy hoặc khởi động lại sẽ dừng các task đang chạy. Lần sau mở Agent View, các Session này sẽ hiển thị là thất bại; sau khi vào, preview hoặc trả lời, có thể tiếp tục cuộc hội thoại trước đó. Những task cần tiếp tục chạy bình thường khi máy offline nên được đưa lên môi trường cloud. Xem chi tiết trong [tài liệu chính thức của Agent View](https://code.claude.com/docs/en/agent-view).

Muốn tìm hiểu thêm sự khác nhau giữa một số cách chạy song song, có thể xem [hướng dẫn chính thức về parallel Agent của Claude Code](https://code.claude.com/docs/en/agents), [Hướng dẫn thực chiến Context Engineering](https://javaguide.cn/ai/agent/context-engineering.html) và [Harness Engineering](https://javaguide.cn/ai/agent/harness-engineering.html).

## Đừng cố định workflow trong thời gian Research Preview

Agent View vẫn đang ở giai đoạn Research Preview, giao diện, nhóm trạng thái và shortcut đều có thể thay đổi. Khi chuẩn bị đưa nó vào workflow của team, sau khi nâng cấp Claude Code nên kiểm tra lại tài liệu chính thức một lần, đừng cố định shortcut hiện tại trong quy chuẩn dài hạn.

Nếu tạm thời không muốn dùng, có thể tắt trong `.claude/settings.json`:

```json
{
  "disableAgentView": true
}
```

## Tổng kết

Tôi thích hiểu Agent View như một task panel của Claude Code hơn. Nó đưa các Session chạy nền rải rác trong terminal vào cùng một danh sách, cho tôi biết Session nào còn đang chạy, Session nào đang chờ trả lời, Session nào đã có thể nghiệm thu.

Task được chia như thế nào, nhiều Agent đồng bộ ra sao, kết quả cuối cùng có đáng tin hay không, vẫn phải dựa vào Worktree, test, CI và con người xử lý.

Với tôi, thay đổi hữu ích nhất của nó vẫn là vấn đề nhỏ ở đầu bài: không cần tìm kiếm xác nhận quyền đã chờ hàng chục phút trong năm sáu tab terminal nữa.
