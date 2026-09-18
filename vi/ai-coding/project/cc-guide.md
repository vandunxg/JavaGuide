---
title: "Sử dụng Claude Code và Codex trong IDEA: Hướng dẫn bắt đầu với CC GUI"
description: CC GUI là một plugin JetBrains mã nguồn mở, cung cấp giao diện trực quan cho Claude Code và OpenAI Codex. Bài viết lấy v0.4.7 làm snapshot, giới thiệu các khả năng thường dùng và giới hạn của plugin như cài đặt, authentication, Diff, Agent và MCP.
category: Lập trình AI thực chiến
head:
  - - meta
    - name: keywords
      content: CC GUI,Claude Code,Codex,IDEA plugin,JetBrains,lập trình AI,Agent,MCP,lập trình trực quan
---

Xin chào mọi người, tôi là Tiểu G. Trước đây tôi đã chia sẻ [thực chiến kết hợp IDEA với plugin Qoder](https://mp.weixin.qq.com/s/vz5A7fQh8WxqVBHscqHzQA), bài viết này sẽ tiếp tục giới thiệu một plugin JetBrains: **CC GUI**.

> **Thông tin phiên bản**: Các tính năng và ảnh chụp màn hình dưới đây được tổng hợp theo CC GUI v0.4.7 (2026-07-24). Plugin được cập nhật khá nhanh, hãy căn cứ vào README của project và giao diện hiện tại để biết cách cài đặt và authentication.

## CC GUI là gì

**CC GUI** (tên ban đầu là Claude Code GUI) là một plugin JetBrains mã nguồn mở sử dụng license MIT, cung cấp giao diện GUI cho Claude Code và OpenAI Codex.

![Giao diện project CC GUI trên GitHub](https://oss.javaguide.cn/github/javaguide/ai/cc-guide/cc-gui-github-project.png)

Địa chỉ project: [zhukunpenglinyutong/jetbrains-cc-gui](https://github.com/zhukunpenglinyutong/jetbrains-cc-gui).

Nếu đã đọc các bài viết trước của tôi, hẳn bạn đã khá quen thuộc với **ACP (Agent Client Protocol)**. Nó định nghĩa một interface tương tác giữa Agent và IDE; việc có thể kết nối thực tế hay không còn phụ thuộc vào protocol version, capability và phương thức authentication mà hai bên triển khai.

Tích hợp Agent tích hợp sẵn trong JetBrains, Agent trong ACP Registry và ACP Agent do người dùng tự cấu hình là các lớp capability khác nhau, không thể gọi chung là “plugin chính thức”. Các Agent có thể sử dụng cũng thay đổi theo IDE version, plugin và account.

CC GUI và ACP là hai hướng khác nhau:

- **Hướng JetBrains/ACP**: sử dụng tích hợp sẵn trong IDE, Registry Agent hoặc ACP Agent tùy chỉnh, tập trung vào việc tái sử dụng khả năng AI Chat, Diff và context của JetBrains; tính năng phụ thuộc vào implementation cụ thể của Agent.
- **Hướng CC GUI**: sử dụng plugin cộng đồng độc lập để bổ sung cho Claude Code và Codex các khả năng GUI như quản lý session, input ảnh, Agent và MCP; phạm vi hỗ trợ phụ thuộc vào version hiện tại của CC GUI.

Hai hướng này không xung đột, bạn có thể chọn theo sở thích.

Lấy v0.4.7 làm snapshot, các khả năng được sử dụng trong bài viết gồm:

- **Hỗ trợ dual engine**: đồng thời kết nối Claude Code và OpenAI Codex, chuyển đổi theo nhu cầu trong phần cài đặt provider.
- **Hội thoại trực quan**: hỗ trợ tham chiếu `@file`, gửi ảnh và rollback hội thoại, trực quan hơn nhiều so với CLI.
- **Agent + MCP**: tích hợp hệ thống Agent và Slash command (như [/loop điều phối](https://mp.weixin.qq.com/s/apkuuxHmC1c6bR0kWhgmUA), [/simplify review code](https://mp.weixin.qq.com/s/Np3oaBmdJAE319wuT7zHBw)), hỗ trợ mở rộng MCP.
- **So sánh Diff**: hiển thị trực tiếp code Diff trong IDEA, hỗ trợ điều hướng file và chuyển tới code.
- **Quản lý session**: lịch sử, tìm kiếm, đánh dấu yêu thích và export.

## Cài đặt và cấu hình

### Bước 1: Cài đặt plugin và SDK

Mở IDEA, vào **Settings → Plugins** (phím tắt `Cmd + ,`), tìm **CC GUI** rồi cài đặt.

![Plugin CC GUI trên IDEA](https://oss.javaguide.cn/github/javaguide/ai/cc-guide/idea-plugin-cc-gui.png)

Sau khi cài đặt xong, bạn có thể tìm thấy entry CC GUI trên thanh công cụ bên phải IDEA và nhấp vào icon để mở.

![Entry CC GUI trên IDEA](https://oss.javaguide.cn/github/javaguide/ai/cc-guide/idea-cc-gui-entry.png)

Lần đầu sử dụng, plugin sẽ nhắc cài đặt Claude Code/Codex SDK. Đây là nền tảng để Agent chạy, hãy nhấp vào đó rồi hoàn tất cài đặt theo giao diện. Thời gian cài đặt phụ thuộc vào network và môi trường máy cục bộ.

![Cài đặt Claude Code/Codex SDK thành công](https://oss.javaguide.cn/github/javaguide/ai/cc-guide/sdk-installed-success.png)

**Gặp màn hình đen?** Một số người dùng gặp màn hình đen khi mở panel CC GUI trên IDEA 2026.1.

Bạn có thể thử xóa cache trình duyệt tích hợp của IDE trước. Nếu vẫn không hiệu quả, một số người dùng trong Issue của project đã thêm các parameter sau vào Help → Edit Custom VM Options để xử lý:

```bash
-Dide.browser.jcef.out-of-process.enabled=false
-Dide.browser.jcef.gpu.disable=true
```

Sau khi thêm, hãy khởi động lại IDEA rồi kiểm tra. Parameter này chỉ là workaround có hiệu lực trong một số môi trường JCEF/card đồ họa, Issue liên quan vẫn chưa đóng tính đến 2026-07-24, không nên xem đây là một bản sửa chắc chắn; parameter cũng có thể ảnh hưởng đến isolation hoặc hardware acceleration của JCEF. Xem thêm: <https://github.com/zhukunpenglinyutong/jetbrains-cc-gui/issues/813>.

### Bước 2: Cấu hình model provider

Nhấp vào phần cài đặt provider và cấu hình theo phương thức authentication mà version hiện tại của plugin hỗ trợ:

- **Đăng nhập OAuth Claude.ai**: cấp quyền sử dụng Claude Code thông qua account Claude.ai có subscription Claude.
- **Anthropic API Key**: API Key lấy từ Anthropic Console và được tính phí riêng theo mức sử dụng API; subscription Claude.ai không tự động cung cấp API Key.
- **Tái sử dụng trạng thái đăng nhập Claude Code CLI**: nếu version hiện tại của plugin hỗ trợ, có thể tái sử dụng trạng thái đăng nhập Claude.ai OAuth hiện có; điều này không có nghĩa là đọc thông tin authentication từ `settings.json`.
- **Import cấu hình Provider cục bộ**: `settings.json` là nơi chứa cấu hình, có thể bao gồm model, endpoint hoặc cài đặt liên quan đến API. Trước khi import, cần xác nhận các field plugin thực sự đọc và cách lưu trữ secret.
- **Import cấu hình cc-switch**: cc-switch là công cụ quản lý provider Claude Code phổ biến trong cộng đồng, CC GUI tương thích với cấu hình của công cụ này, import xong là có thể sử dụng ngay.
- **Endpoint proxy bên thứ ba**: có thể cấu hình endpoint tùy chỉnh, nhưng compatibility, việc xử lý dữ liệu và độ an toàn của secret do dịch vụ proxy quyết định.

Bạn có thể tham khảo [tài liệu authentication chính thức của Claude Code](https://code.claude.com/docs/en/authentication). Codex chính thức hỗ trợ cả đăng nhập ChatGPT và API Key, hai cách này có package và cách tính phí khác nhau; CC GUI có hỗ trợ flow đăng nhập tương ứng hay không thì căn cứ vào version plugin hiện tại, xem thêm [tài liệu authentication của Codex](https://developers.openai.com/codex/auth).

Ảnh chụp màn hình trong bài viết sử dụng cách import cấu hình cc-switch.

![Import trực tiếp cấu hình cc-switch](https://oss.javaguide.cn/github/javaguide/ai/cc-guide/cc-switch-config-import.png)

### Bước 3: Bắt đầu sử dụng

Sau khi cấu hình xong, hãy bắt đầu hội thoại trực tiếp trong panel bên phải. Bạn nên thử một task đơn giản trước, chẳng hạn “hãy phân tích cấu trúc thư mục của project hiện tại”, để cảm nhận khả năng nhận biết context.

Ở đây, chúng ta lấy một scenario thường gặp trong phát triển hằng ngày làm ví dụ: **review code hiện có có tuân thủ quy tắc hay không và sửa hàng loạt các vấn đề**. Làm thủ công việc này cực kỳ nhàm chán: mở file, đối chiếu từng dòng với quy tắc, phát hiện vấn đề, sửa thủ công, rồi chuyển sang file tiếp theo…

CC GUI hỗ trợ **Skill (Slash command)**, có thể tổng hợp một quy trình review cụ thể thành hướng dẫn có thể tái sử dụng. Ví dụ, tôi đã cấu hình một Skill `java-coding-standards`, trong đó có các quy tắc review project Java và Spring Boot.

Ở đây chúng ta lấy project [nền tảng phỏng vấn AI](https://javaguide.cn/zhuanlan/interview-guide.html) làm ví dụ. Khi sử dụng, chỉ cần nhập trực tiếp vào hộp thoại:

```
/java-coding-standards Kiểm tra code trong @infrastructure
```

`/java-coding-standards` load các quy tắc review, còn `@infrastructure` chỉ định phạm vi kiểm tra. Trong lần demo này, Agent đã đọc 14 file Java trong thư mục và xuất ra một báo cáo có cấu trúc:

| Mức độ     | Vấn đề                                                            | File liên quan                | Số lượng |
| ---------- | ----------------------------------------------------------------- | ----------------------------- | -------- |
| Cao        | Log `log.error("xxx: {}", e.getMessage())` làm mất stack trace    | FileHashService               | 3 chỗ    |
| Cao        | BusinessException thiếu ErrorCode                                 | RedisService                  | 1 chỗ    |
| Trung bình | Inline fully qualified class name (`java.util.function.Function`) | InterviewMapper, ResumeMapper | 7 chỗ    |
| Trung bình | Trả về `Map<String, Object>` thay vì DTO chuyên dụng              | InterviewMapper               | 2 chỗ    |
| Thấp       | Resource font không sử dụng try-with-resources                    | PdfExportService              | 1 chỗ    |
| Thấp       | DateTimeFormatter bị tạo lại ở mỗi lần gọi                        | FileStorageService            | 1 chỗ    |

![Báo cáo review có cấu trúc của java-coding-standards](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/java-coding-standards-structured-review-report.png)

Sau khi có báo cáo, bạn có thể yêu cầu AI tạo candidate fix cho từng file, rồi review từng thay đổi và nguyên nhân trong panel Diff.

Lần demo này liên quan đến 9 file và hơn 20 thay đổi, mất chưa đến năm phút từ lúc review đến khi tạo bản sửa và hoàn tất một lần compile verification. Thời gian này chỉ đại diện cho sample hiện tại; kết quả sẽ khác tùy theo quy mô code, model, trạng thái cache và yêu cầu nghiệm thu.

**Giá trị của Skill**: Nó tổng hợp “review gì, review theo các bước nào” thành một entry có thể tái sử dụng, giảm việc phải giải thích lặp lại mỗi lần. Nó có thể tăng tính nhất quán của tiêu chí kiểm tra, nhưng không thể đảm bảo kết quả hoàn toàn giống nhau giữa các model và context khác nhau; standard của team vẫn nên được đặt trong rule có thể version hóa, static check và quy trình Review.

Bạn có thể đọc hai bài viết sau của tác giả để xem đề xuất Skills hữu ích cho Vibe Coding và giải đáp các câu hỏi thường gặp về Skills:

1. [Danh sách chọn AI Coding Skills: Làm rõ yêu cầu, TDD, review code và thiết kế UI](https://javaguide.cn/ai-coding/practices/programmer-essential-skills.html)
2. [Agent Skills là gì? Khác Prompt và MCP chính xác ở đâu?](https://mp.weixin.qq.com/s/5iaTBH12VTH55jYwo4wmwA)

## Tính năng tích hợp của CC GUI

CC GUI còn tích hợp tính năng thống kê sử dụng, cho phép xem rõ mức tiêu thụ Token, thống kê chi phí và phân tích xu hướng sử dụng.

![Thống kê sử dụng CC GUI](https://oss.javaguide.cn/github/javaguide/ai/cc-guide/cc-gui-usage-stats.png)

Ngoài ra còn hỗ trợ Commit AI, intelligent agent tùy chỉnh, quản lý thư viện prompt, thêm MCP server và các tính năng khác.

![Commit AI của CC GUI](https://oss.javaguide.cn/github/javaguide/ai/cc-guide/cc-gui-commit-ai.png)

Bạn cũng có thể xem các message cũ, đồng thời hỗ trợ tìm kiếm và xóa:

![Message cũ của Claude Code](https://oss.javaguide.cn/github/javaguide/ai/cc-guide/claude-code-history.png)

## Nên chọn CC GUI hay Qoder?

Hai plugin này có định vị khác nhau, hãy cùng so sánh đơn giản:

| Tiêu chí             | CC GUI                                            | Qoder                                        |
| -------------------- | ------------------------------------------------- | -------------------------------------------- |
| **Định vị**          | Lớp GUI cho Claude Code / Codex                   | AI Coding Agent độc lập                      |
| **Mã nguồn mở**      | License MIT, hoàn toàn mã nguồn mở                | Mã nguồn đóng, do Alibaba phát triển         |
| **Model**            | Claude Code + Codex, phạm vi hỗ trợ tùy version   | Model tích hợp và model có thể chọn hiện tại |
| **Context**          | Tham chiếu `@file` + input ảnh                    | `@database` + `@file`                        |
| **Scenario phù hợp** | Muốn sử dụng workflow CLI hiện có trong JetBrains | Muốn sử dụng Agent tích hợp tất cả trong một |
| **Tối ưu cho Java**  | Phổ dụng                                          | Tối ưu khá tốt cho hệ sinh thái Java         |

**Đề xuất của tôi:**

- **Đã có workflow Claude Code hoặc Codex** → có thể đánh giá CC GUI, nhưng trước tiên hãy xác nhận phương thức authentication và mapping tính năng mà plugin hỗ trợ; GUI không đảm bảo kế thừa đầy đủ mọi capability của CLI
- **Muốn dùng ngay, không muốn mất công cấu hình API** → chọn Qoder, đăng ký là có thể sử dụng
- **Cài cả hai cũng được** → chúng không xung đột, có thể chuyển đổi theo scenario

## Tổng kết

Giá trị cốt lõi của CC GUI là **hoàn thiện workflow trực quan cho người dùng JetBrains**. Nó cố gắng đưa các thao tác vốn phân tán trong terminal, editor, công cụ chụp màn hình và file manager về thực hiện tại một nơi trong IDE.

Nếu bạn chủ yếu sử dụng JetBrains và muốn quản lý session Claude Code hoặc Codex trong IDE, có thể dùng thử CC GUI trong project test, sau đó quyết định có đưa vào workflow hằng ngày hay không dựa trên authentication, compatibility tính năng và yêu cầu bảo mật của team.
