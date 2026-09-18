---
title: Trải nghiệm oh-my-pi, Agent coding AI mã nguồn mở trên terminal
description: Giới thiệu các năng lực cốt lõi của oh-my-pi, gồm cơ chế patch Hashline, tích hợp LSP và DAP, công cụ tích hợp, định tuyến nhiều model, cách cài đặt, cấu hình và đề xuất sử dụng.
category: Thực chiến AI coding
head:
  - - meta
    - name: keywords
      content: oh-my-pi,omp,AI coding,Agent coding AI trên terminal,thay thế Claude Code,OpenCode,Codex CLI,Hashline,LSP,DAP,định tuyến nhiều model
---

Sau khi xác nhận với một người bạn ở Alibaba, từ ngày 10 tháng 7, Alibaba sẽ đưa Claude Code vào danh sách phần mềm có rủi ro cao và khuyến nghị nhân viên nội bộ sử dụng Qoder để thay thế.

Không bàn sâu chuyện này.

Mặc dù hãng A thường làm chuyện không ra gì, model Claude và Claude Code thực sự làm rất tốt. So với các sản phẩm cùng loại, nó vẫn là sản phẩm ổn định nhất. Dù sao đây cũng là dự án thương mại, đội ngũ đều rất giỏi và nhịp phát hành sản phẩm rất nhanh.

Các dự án cùng loại và khá nổi tiếng gồm OpenCode, Codex CLI, Cline, Trae, Qoder; trước đây DeepSeek TUI sau đó còn đổi tên thành CodeWhale.

Vài ngày trước, một người bạn trong nhóm gửi một **oh-my-pi** GitHub link và nói gần đây dùng khá dễ chịu.

Ban đầu tôi cũng không để tâm lắm, suy nghĩ thầm: **Lại một Agent trên terminal? Nó khác Claude Code, OpenCode và Codex CLI ở đâu?**

Sau vài ngày sử dụng, thái độ của tôi đã thay đổi.

## Nó là gì

oh-my-pi là một Agent coding AI mã nguồn mở trên terminal.

Sau khi cài đặt thành công, bạn thực thi lệnh `omp` trong thư mục dự án, rồi có thể yêu cầu nó đọc code, sửa code, chạy lệnh, giải thích lỗi và tạo nội dung commit.

Điều này khá giống các công cụ như Claude Code và Codex CLI.

Khác biệt chủ yếu nằm ở **tool layer**.

LSP, DAP, Hashline, browser, GitHub, sub-Agent, định tuyến nhiều model và nhiều thứ khác đều được đưa vào terminal.

Ví dụ khi rename function, nó có thể dùng language server để tra cứu references, bớt phải đoán mò bằng `grep`; khi debug một crash, nó có thể vào debugger để xem stack frame và variables; khi xem PR, nó cũng có thể coi PR là một resource có thể đọc.

Ngoài ra còn một điểm nhỏ rất chủ quan: tôi khá thích terminal UI của nó, rất hợp gu của tôi.

Điều này không phải năng lực cốt lõi, nhưng những người ngày nào cũng làm việc trước terminal chắc sẽ hiểu: giao diện dễ nhìn thực sự ảnh hưởng đến tâm trạng.

## Hashline

Nhiều Agent sửa file, thực tế vẫn dùng `old_string -> new_string`.

Đầu tiên đọc một đoạn file, sau đó yêu cầu model nhắc lại nội dung gốc, rồi tool dùng đoạn nội dung này để tìm và thay thế.

Có lẽ mọi người đều từng gặp vấn đề với cách này.

Thiếu một dấu cách, thừa một dòng mới, thụt lề lệch một chút là patch không tìm được vị trí. Phiền hơn là bạn vừa sửa file thủ công, model lại dùng context cũ để sửa; nội dung mới và cũ trộn lẫn vào nhau là hiện trường lập tức rối tung.

Trong tool `edit` của oh-my-pi có một thứ tên là **Hashline**.

`@oh-my-pi/hashline` mô tả nó là một compact, line-anchored patch language. Đại khái là khi đọc file, mỗi dòng sẽ kèm một content hash; khi model sửa file, nó thực hiện thay đổi xoay quanh hash, bớt phải nhắc lại toàn bộ nội dung gốc.

![Ảnh chụp giải thích Hashline chính thức của oh-my-pi](https://oss.javaguide.cn/github/javaguide/ai/coding/oh-my-pi/oh-my-pi-hashline-doc.png)

Nếu file thay đổi giữa chừng, hash không khớp thì patch sẽ bị từ chối trước.

Điều này không chỉ nhằm làm patch ngắn hơn, mà quan trọng hơn là bổ sung một lớp định vị và kiểm tra ổn định. Model không thể mãi nhắc lại nội dung gốc chính xác từng chữ, nên tool layer thêm một lớp bảo vệ trước.

Benchmark chính thức đề cập rằng Grok 4 Fast tạo ít hơn 61% token trong các task cùng loại. Tôi chưa tái hiện thí nghiệm này, nên ở đây chỉ xem đó là số liệu từ phía dự án và chỉ tham khảo xu hướng.

So với việc tiết kiệm token, tôi quan tâm hơn đến việc patch lỗi có được chặn sớm hay không. Khi thực sự dùng trong dự án, tránh được một lần sửa loạn quan trọng hơn tiết kiệm vài trăm token.

## Nơi giống IDE nhất

Điểm khiến oh-my-pi giống IDE nhất là nó đưa trực tiếp hai nhóm năng lực thường dùng của IDE là LSP và DAP vào tool surface của Agent.

Phía chính thức chia chúng thành hai loại:

- `lsp` phụ trách diagnostics, navigation, symbols, renames, code actions, raw requests;
- `debug` phụ trách breakpoints, stepping, threads, stack, variables trong session DAP.

Hai thuật ngữ này nghe khá low-level, nhưng đặt vào việc coding hằng ngày thì thực ra rất dễ hiểu.

**LSP phụ trách trả lời cấu trúc code là gì.** Ví dụ một function được định nghĩa ở đâu, được những nơi nào tham chiếu, file hiện tại có diagnostics nào, một lần rename sẽ ảnh hưởng đến những import và re-export nào.

Trước đây khi Agent muốn đổi tên function, nhiều lúc chỉ có thể `grep` trước, rồi để model phán đoán các kết quả khớp đó có phải cùng một symbol hay không. Quá trình này rất dễ lẫn comment, string và variable cùng tên. Đặc biệt trong các dự án TypeScript, khi barrel export, path alias và re-export tăng lên, text search thuần túy sẽ bắt đầu không đủ dùng.

Nếu nó có thể đi qua LSP để thực hiện rename, references và diagnostics, ít nhất thứ nhận được là các quan hệ code dưới góc nhìn của language server. Model vẫn có thể phán đoán sai, nhưng không còn hoàn toàn đứng ngoài text để đoán nữa.

Điểm này khá quan trọng với tôi.

Khi để Agent viết code, điều tôi lo nhất là nó nghiêm túc đoán quan hệ gọi, đoán sai nhưng vẫn tiếp tục viết. LSP ít nhất cung cấp cho nó một bản đồ gần với IDE hơn.

**DAP phụ trách một việc khác: runtime.**

Trước đây khi để Agent debug, nó rất dễ đi theo một quy trình: thêm log, chạy, xem output, rồi thêm log. Cách này dĩ nhiên hữu ích, nhưng khi gặp crash native, service Go bị hang hoặc process Python bị treo thì chỉ dựa vào log sẽ rất chậm.

Với DAP, ít nhất nó có thể đặt breakpoint, thực hiện từng bước, xem thread, xem call stack và đọc variable. Thứ nhận được ở đây là trạng thái runtime, không chỉ là kết quả text matching.

Dĩ nhiên điều này không có nghĩa là nó chắc chắn sửa được bug. Chỉ là khi debug, thứ nó quan sát bắt đầu gần với thứ một developer thực sự sẽ xem: trước tiên xem đang dừng ở đâu, sau đó xem variable thay đổi thế nào, cuối cùng mới quyết định viết patch ra sao.

Vì vậy tôi thích hiểu phần này là “năng lực IDE trong terminal”. Nó không cùng một chiều với số lượng tool ở phần sau.

Các tool `eval`, `task`, `browser`, `github` ở phần sau giống workbench của Agent hơn; LSP và DAP mới là phần cốt lõi khiến nó giống IDE nhất.

## Tool rất phong phú

Tool tích hợp của đa số Agent khá tiết chế: đọc file, tìm text, sửa file và chạy lệnh. Những việc nặng hơn thường được giao cho MCP server.

omp đi theo hướng khác. Tổng cộng có 32 tool tích hợp, trông hơi nặng nhưng quả thực có vài tool khá tiêu biểu.

Đầu tiên là `eval`. Nó tích hợp sẵn Python và Bun JS kernel thường trú, không phải sandbox dùng một lần rồi bỏ. Quan trọng hơn, hai kernel này còn có thể gọi ngược lại các tool của chính omp, chẳng hạn `read`, `search`, `task`. Agent có thể đọc CSV trong Python cell, sau đó chuyển sang JavaScript cell để xử lý dữ liệu mà không cần rời session.

Các tool điều phối multi-Agent hiện tại là `task`, `hub`, `todo` và `ask`, không phải `task` / `irc` trong tài liệu cũ. `task` phụ trách fan-out sub-Agent; mỗi worker có thể dùng tool surface riêng và cũng có thể được cô lập trong worktree riêng; `hub` dùng cho việc cộng tác và nhắn tin giữa các Agent đang chạy, `todo` quản lý trạng thái task, `ask` dùng để gửi câu hỏi có cấu trúc. Các field cụ thể thay đổi khá nhanh, nên hãy lấy tool description hiện tại làm chuẩn.

`browser` dựa trên Puppeteer, cũng cung cấp xử lý liên quan đến stealth và có thể điều khiển ứng dụng Electron thông qua CDP. Stealth chỉ có thể giảm một phần dấu hiệu tự động hóa phổ biến, không đảm bảo website nhận diện nó như người dùng thông thường, càng không thể vượt qua điều khoản website, giới hạn đăng nhập hoặc chính sách chống tự động hóa.

`github` là phần tôi thích hơn. Nó không bắt model phải học thêm một đống `gh_issue_view`, `gh_pr_view`, `gh_search`, mà coi PR như một path của file system để đọc. Cấu trúc nhận được từ `read pr://1428` cũng theo cùng một cách tư duy với `read src/foo.ts`; `search` cũng có thể duyệt diff như duyệt directory. Abstraction này còn mở rộng thành các internal scheme như `pr://`, `issue://`, `agent://`, `skill://`, `rule://`, `conflict://`.

Tôi tiện tay thử với một issue của JavaGuide. Nó đọc issue trước, sau đó tìm Markdown tương ứng trong repository, rồi tiếp tục đọc theo image link.

![oh-my-pi đọc GitHub issue và lần theo file trong repository](https://oss.javaguide.cn/github/javaguide/ai/coding/oh-my-pi/oh-my-pi-github-issue-read.png)

Còn có `advisor`, có thể gắn một reviewer model. Mỗi vòng nó xem output của main Agent, sau đó inject inline các nhắc nhở trở lại. Nó chạy context và model riêng, chuyên tìm những thứ main Agent bỏ sót. Thiết kế này hơi giống có một người chỉ chuyên bắt lỗi ngồi bên cạnh.

Xét riêng lẻ, các tool này chưa chắc đều mới mẻ; nhưng khi đặt trong cùng một tool surface thì có điểm khác. Đọc file local, đọc PR và đọc kết quả của sub-Agent đều cố gắng quy về cùng một hành động “đọc resource”, để model bớt phải học một đống interface kỳ lạ.

Nhưng nhiều tool cũng có mặt trái: routing layer sẽ phức tạp hơn và cơ hội gọi nhầm tool cũng tăng lên. Vì vậy tôi không khuyến nghị lần đầu đã cấu hình tất cả.

Nhiều model, nhiều tool và nhiều memory nghe rất hấp dẫn, nhưng cấu hình, chi phí và quyền hạn cũng tăng theo. Đặc biệt với các tool như `bash`, `write`, `edit`, `browser`, `github`, `ssh`, trước khi bật tốt nhất hãy nghĩ rõ chúng có thể chạm đến những gì.

Một phần tool mặc định bị tắt, danh sách sẽ thay đổi theo version. Bạn nên xem trực tiếp `/tools` hoặc help information hiện tại, không nên duy trì một danh sách tĩnh trong bài viết.

Nếu thực sự muốn thu hẹp tool surface, có thể dùng `--tools read,edit,bash,...` để chỉ expose một phần. Năng lực ẩn hiện được expose theo nhu cầu thông qua cơ chế resource discovery `xd://`, không còn dùng `search_tool_bm25` trong tài liệu cũ.

Chiến lược mặc định này là đúng. Điểm phiền phức nhất của Agent trên terminal thường không phải là thiếu năng lực, mà là sau khi cấp quá nhiều quyền thì chính mình cũng không nhớ nó có thể chạm đến đâu.

## Nhà cung cấp nhiều model và định tuyến theo role

Ban đầu tôi không nghĩ danh sách model lại phong phú đến vậy.

Không chỉ có ba nhà cung cấp nước ngoài phổ biến là OpenAI, Anthropic và Gemini; các gói coding subscription như Cursor, Copilot, Kimi Code, Moonshot, Tongyi, Qwen Portal, GLM, Xiaomi MiMo, Qianfan cũng có thể được tìm thấy. Về local model thì có Ollama, LM Studio, llama.cpp, vLLM, LiteLLM và nhiều lựa chọn khác.

Ngoài ra, nó còn hỗ trợ định tuyến theo 5 role: `default`, `smol`, `slow`, `plan`, `commit`.

Tôi sẽ coi `default` là model chính, thường dùng nó để đọc code, sửa code và hỏi đáp. `smol` phù hợp hơn để giao cho sub-Agent thực hiện những việc nhỏ như kiểm tra hàng loạt file, quét thông tin; chỉ cần rẻ hơn và nhanh hơn một chút, không kỳ vọng nó đưa ra phán đoán phức tạp.

Khi thực sự gặp phán đoán kiến trúc, bug khó hoặc suy luận với context dài, hãy để `slow` dùng model mạnh hơn nhưng đắt hơn. `plan` dùng để suy nghĩ trước xem cần sửa những file nào và chia các bước ra sao; còn `commit` dành cho các việc văn bản theo format cố định như changelog và commit message.

![oh-my-pi đặt cùng một model thành các role khác nhau trong model panel](https://oss.javaguide.cn/github/javaguide/ai/coding/oh-my-pi/oh-my-pi-model-role-actions.png)

Như vậy hội thoại hằng ngày, sub-Agent, suy luận chuyên sâu, lập kế hoạch và commit message không phải chen chúc trong một model. Nó giống phân luồng theo chi phí và chất lượng hơn, không khiến bản thân model tự nhiên mạnh lên. Trong main session có thể lần lượt chuyển bằng `Ctrl+P`, hoặc dùng `/model` để đổi thủ công.

Tôi sẽ không cấu hình tất cả ngay từ đầu. Trước tiên để `default` chạy ổn định, sau đó mới cân nhắc `smol` và `slow`. Fallback chain, model scope theo path và xoay vòng nhiều key đều trông rất hấp dẫn, nhưng bật hết ngay ngày đầu sẽ rất khó phán đoán khi có vấn đề thì tầng nào đang gặp trục trặc.

## Bắt đầu sử dụng

Cài đặt chỉ cần một lệnh, rất đơn giản.

macOS / Linux:

```sh
curl -fsSL https://omp.sh/install | sh
```

Homebrew:

```sh
brew install can1357/tap/omp
```

Bun:

```sh
bun install -g @oh-my-pi/pi-coding-agent
```

Windows PowerShell:

```powershell
irm https://omp.sh/install.ps1 | iex
```

Dự án yêu cầu `bun >= 1.3.14`, `package.json` ở root repository cũng ghi `packageManager: bun@1.3.14`. Nếu cài bằng Bun, hãy kiểm tra version trước để không mắc kẹt ở môi trường.

Sau khi cài xong, chỉ cần thử với một dự án không quá quan trọng.

```sh
cd your-project
omp
```

Lần đầu vào chương trình sẽ chưa bắt đầu chat ngay mà trước tiên yêu cầu bạn thực hiện setup.

Trước tiên chọn provider muốn đăng nhập. Có thể kết nối nhiều provider, chẳng hạn ChatGPT Plus/Pro, Anthropic, Z.AI, Kimi Code, OpenRouter, Copilot và Cursor đều sẽ được liệt kê. Provider đã được cấu hình bằng environment variable cũng sẽ hiển thị trực tiếp là logged in.

![Chọn model provider khi oh-my-pi khởi động lần đầu](https://oss.javaguide.cn/github/javaguide/ai/coding/oh-my-pi/oh-my-pi-setup-provider-login-kimi.png)

![Trang quyền lợi thành viên Kimi Code](https://oss.javaguide.cn/github/javaguide/ai/coding/oh-my-pi/oh-my-pi-kimi-code-home.png)

Sau đó chuyển sang Web search và chọn backend search nào được `web_search` ưu tiên sử dụng. Dự án hiện đã mở rộng đến khoảng 25 backend; việc liệt kê tĩnh tên các backend sẽ nhanh chóng lỗi thời. Khi chọn `Auto`, nó sẽ chọn trong các backend đã cấu hình; ở chế độ thủ công, hãy lấy Setup page hiện tại làm chuẩn.

![Chọn Web search provider khi oh-my-pi khởi động lần đầu](https://oss.javaguide.cn/github/javaguide/ai/coding/oh-my-pi/oh-my-pi-setup-web-search.png)

Không cần mất quá nhiều thời gian cân nhắc bước này. Trước tiên làm cho một model chính và một search provider chạy được sẽ ổn định hơn việc kết nối tất cả account ngay từ đầu.

Sau khi cấu hình xong và quay lại main interface, model ở đây đã được chọn sẵn, bên trái hiển thị `DeepSeek V4 Flash`.

![oh-my-pi tự động chọn model mặc định sau khi khởi động](https://oss.javaguide.cn/github/javaguide/ai/coding/oh-my-pi/oh-my-pi-welcome-default-model.png)

Ban đầu tôi hơi ngẩn ra: hình như lúc nãy tôi không tự chọn DeepSeek, sao nó lại tự cấu hình xong?

Vì vậy tôi tiện thể hỏi nó. Giải thích đại khái của nó là: oh-my-pi tích hợp sẵn một model catalog; khi khởi động, nó lần lượt tìm credential khả dụng, chẳng hạn command-line argument, `models.yml`, key / OAuth đã lưu từ `/login` trước đó, environment variable và một số file `.env`. Chỉ cần phát hiện các variable như `DEEPSEEK_API_KEY` khớp được, nó sẽ đánh dấu model tương ứng trong provider đó là khả dụng, rồi tự động chọn một model ban đầu.

![oh-my-pi giải thích lý do model tự động được cấu hình](https://oss.javaguide.cn/github/javaguide/ai/coding/oh-my-pi/oh-my-pi-model-auto-config-reason.png)

Sau này muốn đổi model cũng không cần restart, chỉ cần dùng `/model`. Nó chỉ hiển thị các model đã có credential khả dụng, phía trên còn có thể chuyển tab theo provider. Ở đây tôi có thể thấy các entry DeepSeek, Z.AI, Ollama, LM Studio và llama.cpp.

![oh-my-pi chuyển model bằng lệnh model](https://oss.javaguide.cn/github/javaguide/ai/coding/oh-my-pi/oh-my-pi-model-switch.png)

Nếu muốn xem version hiện tại có những command và parameter nào, chỉ cần chạy:

```sh
omp --help
```

Nếu trước đây bạn từng dùng các tool như Claude Code, Codex CLI, Cursor, Windsurf, Gemini và Cline, nó cũng sẽ đọc rules, skills và MCP servers đã có trên disk. Các directory như `.claude`, `.cursor`, `.windsurf`, `.gemini`, `.codex`, `.cline`, `.github/copilot`, `.vscode` đều nằm trong phạm vi nó sẽ xem.

## Tổng kết

Đặc điểm của oh-my-pi chủ yếu tập trung ở tool layer: Hashline dùng để tăng độ ổn định khi định vị patch, LSP và DAP cung cấp cấu trúc code cùng trạng thái runtime, còn các tool `github`, `browser`, `task`, `eval` đưa nhiều resource khác nhau vào cùng một session.

Nhiều tool cũng đồng nghĩa với phạm vi quyền hạn lớn hơn. Các năng lực như `github`, `browser`, `memory`, `ssh` nên được bật từng phần theo task; khi liên quan đến account, private message, thao tác ghi repository và remote host, hãy xác nhận trước phạm vi quyền hạn, audit record và điều khoản dịch vụ.

Với những người đã quen terminal workflow và muốn thử nghiệm model, tool cùng quyền hạn, có thể lấy một dự án cá nhân để thử. Nếu chỉ muốn một tool ổn định, ít phải cấu hình và mở lên là dùng được thì Claude Code vẫn đỡ tốn công hơn.

Điểm oh-my-pi hấp dẫn tôi nhất hiện nay là nó đã đẩy tool layer của Agent mã nguồn mở tiến thêm một bước. Nhiều thứ, tham vọng lớn, điểm nổi bật cũng rất rõ ràng, nhưng cần từng bước thử nghiệm để xây dựng niềm tin.

Địa chỉ dự án: [https://github.com/can1357/oh-my-pi](https://github.com/can1357/oh-my-pi)

Trang web chính thức: [https://omp.sh](https://omp.sh)
