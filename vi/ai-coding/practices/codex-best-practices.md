---
title: "Hướng dẫn sử dụng Codex: cấu hình, AGENTS.md và quy trình Agentic"
description: Kết hợp tài liệu chính thức của OpenAI và thực tiễn cộng đồng Codex CLI để giải thích rõ về mô tả nhiệm vụ, giai đoạn lập kế hoạch, AGENTS.md, config.toml, kiểm soát quyền, MCP, Skills, Subagents, Hooks và Scheduled Tasks.
category: Thực hành lập trình AI
head:
  - - meta
    - name: keywords
      content: OpenAI Codex,Codex CLI,Lập trình AI,AGENTS.md,Agent Skills,MCP,Subagents,Hooks,Scheduled Tasks,Phát triển có AI hỗ trợ
---

Xin chào, tôi là G. Trước đây tôi từng viết bài [Hướng dẫn sử dụng Claude Code: cấu hình, workflow và kỹ thuật nâng cao](./claudecode-tips.md). Sau khi đăng bài, có bạn hỏi: Claude Code đã được trình bày nhiều như vậy, vậy dùng Codex thế nào để ổn định hơn?

Khi mới bắt đầu dùng Codex, kỳ vọng của tôi thực ra không cao.

Xét theo tên gọi, rất dễ nghĩ nó chỉ là một trợ lý dòng lệnh viết code giỏi hơn. Sau một thời gian sử dụng thực tế, cảm nhận lại khác: Codex giống một trợ lý kỹ thuật có thể tự đọc repository, sửa file, chạy command rồi quay lại bàn giao. Nó không phù hợp để chỉ dùng bổ sung vài dòng code, mà phù hợp hơn với những nhiệm vụ có mục tiêu rõ ràng, có thể verify và có phạm vi được nêu cụ thể.

Nhưng điều này có một tiền đề.

Bạn phải sắp xếp workbench trước: phạm vi nhiệm vụ, quyền hạn, quy tắc dự án và tiêu chuẩn nghiệm thu đều phải nói rõ từ trước.

Mô tả nhiệm vụ quá mơ hồ thì nó sẽ đoán khắp nơi; cấp quyền quá rộng thì nó có thể tiện tay thực hiện những hành động bạn chưa cho phép; viết `AGENTS.md` thành bài giới thiệu dự án thì mỗi lượt nó vẫn phải hiểu lại repository; không đưa tiêu chuẩn nghiệm thu thì nó rất dễ dừng ở mức “trông như đã sửa xong”.

Bài viết này không định giới thiệu Codex theo lịch sử phát hành sản phẩm, cũng không xoay quanh tên một model nào. Model, gói dịch vụ và chi tiết command thay đổi rất nhanh, viết cố định sẽ dễ lỗi thời. Điều đáng giữ lại hơn là một số kinh nghiệm khá bền vững trong dự án thực tế: giao nhiệm vụ thế nào, khi nào nên vào giai đoạn lập kế hoạch (Plan), nên đặt gì trong `AGENTS.md`, `config.toml` quản lý gì, phân tầng quyền hạn, Rules và Hooks ra sao, còn MCP, Skills, Subagents và Scheduled Tasks lần lượt phù hợp với trường hợp nào.

Trước hết cần nói rõ phạm vi: bài viết này chủ yếu dành cho việc sử dụng hằng ngày **Codex CLI + Codex App**. Những command và khả năng thấy được ở IDE Extension, Web/Cloud có thể không hoàn toàn giống nhau.

## Đừng chỉ viết một câu cho nhiệm vụ

Nhiều người lần đầu giao nhiệm vụ cho Codex sẽ viết như sau:

```text
Hãy tối ưu logic đăng nhập giúp tôi.
```

Câu này với con người còn chưa đủ, đương nhiên với Codex cũng chưa đủ. Logic đăng nhập nằm ở đâu? Tối ưu về performance, khả năng đọc, security hay Bug trên production? Những file nào không được động vào? Sau khi sửa thì dùng gì để chứng minh nó thực sự tốt hơn?

Trong best practice chính thức của OpenAI có một framework rất thực tế: Goal, Context, Constraints, Done when. Viết theo cách thường ngày có nghĩa là nói rõ “cần làm gì, xem ở đâu, không được đụng vào gì, đạt đến mức nào thì tính là xong”. Done when không nên chỉ viết “chức năng hoạt động bình thường”, tốt nhất hãy viết rõ các bằng chứng có thể verify như test, build, lint, screenshot, log hoặc command output.

Ví dụ cùng là sửa vấn đề đăng nhập, tôi sẽ viết lại như sau:

```text
Mục tiêu: Sửa vấn đề refresh token vẫn còn hiệu lực nhưng refresh thất bại sau khi session của người dùng hết hạn.
Bối cảnh: Tập trung đọc src/auth, src/session và AuthControllerTest.
Ràng buộc: Không sửa cấu trúc bảng database, không thêm dependency mới, giữ nguyên format trả về Result<T> hiện tại.
Tiêu chuẩn hoàn thành: Bổ sung một test tái hiện được vấn đề, chạy các test liên quan sau khi sửa implementation, đồng thời báo cáo command và kết quả.
```

Cách này có thể giảm không gian suy đoán của Codex.

Nhiệm vụ nhỏ có thể đơn giản hơn. Ví dụ sửa một đoạn text, bổ sung một log, thống nhất tên một parameter nào đó thì chỉ cần nói rõ mục tiêu là đủ. Nhưng một khi nhiệm vụ liên quan đến quyền hạn, thanh toán, trạng thái order, data migration, concurrency hay compatibility, tốt nhất đừng tiết kiệm vài dòng mô tả. Bạn viết thêm 2 phút ở phía trước sẽ giảm rất nhiều diff kỳ lạ phải xem về sau.

Một thói quen khác cũng rất hữu ích: **đưa tài liệu gốc cho Codex, đừng chỉ đưa phán đoán của mình.**

Ví dụ với lỗi trên production, đừng chỉ nói “chắc là cache chưa được clear”. Hãy đưa stack trace, request parameter, các bước reproduce, test thất bại và output của browser console để nó tự định vị. Nếu bạn kết luận trước, nó rất dễ đi theo phỏng đoán của bạn, cuối cùng sửa một vấn đề configuration thành vấn đề business logic.

## Nhiệm vụ phức tạp hãy để nó khảo sát trước

Codex có thể sửa code trực tiếp, nhưng không có nghĩa lần nào cũng nên sửa ngay.

Hiện giờ tôi sẽ xem xét risk của nhiệm vụ trước. Những việc như sửa text, bổ sung field hay thêm một lớp bảo vệ null rõ ràng thì cứ để nó làm trực tiếp. Nó đọc file, sửa file rồi chạy test, hiệu suất rất cao.

Một loại nhiệm vụ khác lại không như vậy. Ví dụ bạn muốn sửa state machine của order, tách một function hơn 600 dòng, hoặc điều tra một timeout xảy ra ngẫu nhiên. Chính bạn còn chưa hiểu hoàn toàn call chain, nếu lúc này để Codex lập tức viết code thì rất dễ càng sửa càng rối.

Với loại nhiệm vụ này, trước tiên tôi sẽ để nó vào giai đoạn lập kế hoạch:

```text
Trước tiên hãy vào giai đoạn lập kế hoạch, không sửa file.
Đọc src/payment, src/order và các test liên quan để hiểu rõ call chain từ lúc thanh toán thành công đến khi trừ inventory.
Hãy xuất ra các file quan trọng, flow hiện tại, các điểm có thể sửa, risk và command verify được đề xuất.
```

Sau khi nó đọc xong repository, hãy để nó tách kế hoạch:

```text
Dựa trên phân tích vừa rồi, hãy đưa ra một kế hoạch theo từng giai đoạn.
Mỗi giai đoạn cần ghi rõ sẽ sửa file nào, bổ sung test nào và verify ra sao.
Không bắt đầu implementation, hãy chờ tôi xác nhận.
```

Quy trình này chậm trong 10 phút đầu, nhưng nhanh hơn về sau.

Điểm thực sự khó của project cũ thường không phải là viết một đoạn code nào đó khó, mà là logic compatibility lịch sử, feature flag, fallback của configuration và các ranh giới không dám động vào bị trộn lẫn với nhau. Giá trị của giai đoạn lập kế hoạch (Plan) là trước tiên lôi những thứ này ra.

Output của giai đoạn lập kế hoạch chỉ là phương án ứng viên, không phải sự thật cuối cùng. Với thay đổi có risk cao, con người vẫn phải xác nhận call chain quan trọng, ranh giới transaction và compatibility.

Tuy nhiên cũng đừng coi giai đoạn lập kế hoạch (Plan) là nghi thức. Khi nhiệm vụ đủ nhỏ và nghiệm thu đủ rõ ràng, thực hiện trực tiếp lại tốt hơn. Có một quan điểm trong cộng đồng mà tôi khá đồng tình: Codex thông thường kết hợp với nhiệm vụ nhỏ sẽ dễ tạo output ổn định hơn workflow phức tạp.

## AGENTS.md rất quan trọng

### Đừng viết nó thành README thứ hai

Nó hơi giống `CLAUDE.md` trong Claude Code, đều là file instruction cấp project dành cho Agent. Nói thẳng hơn, `AGENTS.md` là một **hướng dẫn công việc cho Agent**: cho Codex biết project này khởi động và test thế nào, directory nào không được đụng vào, sau khi sửa cần đưa ra bằng chứng gì.

![CLAUDE.md và AGENTS.md trong project phân tích cổ phiếu bằng multi-agent](https://oss.javaguide.cn/github/javaguide/ai/coding/claude-agents-md.png)

Tuy nhiên vai trò của hai file không hoàn toàn giống nhau.

`CLAUDE.md` là entry point dành riêng cho Claude Code, chủ yếu do Claude Code đọc; `AGENTS.md` là một format file instruction mở dành cho coding agents và được OpenAI Codex hỗ trợ chính thức. Các tool khác có đọc hay đọc thế nào thì phải xem tài liệu tương ứng, đừng mặc định mọi Agent đều load theo cùng một bộ quy tắc.

Nếu repository đã có `AGENTS.md`, thường không cần duy trì thêm một `CLAUDE.md` có nội dung gần như giống hệt. Có thể để `CLAUDE.md` import `AGENTS.md`, sau đó bổ sung các yêu cầu riêng của Claude Code:

```markdown
@AGENTS.md

## Instruction riêng của Claude Code

- Dùng plan mode để xử lý thay đổi trong `src/billing/`.
```

Như vậy chỉ cần duy trì quy tắc nền tảng ở một nơi, cả Claude Code và Codex đều có thể dùng lại. Ngược lại, nếu ban đầu team chỉ có `CLAUDE.md` mà giờ muốn các tool như Codex, Cursor cũng đọc được cùng một bộ quy ước, có thể tách phần dùng chung vào `AGENTS.md`, còn command riêng của Claude Code để lại trong `CLAUDE.md`.

Tôi đề xuất `AGENTS.md` chỉ chứa thông tin mà Agent thực sự sẽ dùng:

- Quy tắc Codex dễ đoán sai
- Quy ước không thể đọc ra từ code
- Tiêu chuẩn team bắt buộc tuân thủ
- Version của tech stack, command thường dùng, cấu trúc hệ thống, các điểm cần lưu ý của project

### Đặt theo phân tầng

Khi khởi động, Codex sẽ xây dựng một instruction chain. Theo thứ tự discovery trong tài liệu chính thức hiện tại: trước tiên đọc `AGENTS.override.md` trong Codex home, nếu không có thì đọc `AGENTS.md`; sau đó đi từ root của project đến directory hiện tại. Mỗi directory theo thứ tự `AGENTS.override.md`, `AGENTS.md`, fallback filenames sẽ đọc nhiều nhất một file. Instruction càng gần working directory hiện tại thì càng nằm về sau và càng dễ ảnh hưởng đến nhiệm vụ lần này.

`AGENTS.override.md` phù hợp để tạm thời override `AGENTS.md` cùng directory. Nếu chỉ muốn thay đổi một rule trong thời gian ngắn mà không muốn động vào file nền tảng, có thể dùng nó.

Ngoài ra còn một giới hạn không quá nổi bật nhưng rất thực tế: `project_doc_max_bytes` mặc định giới hạn kích thước instruction của project sau khi Codex merge, mặc định chính thức là 32 KiB. Dù có thể tăng lên, cũng không nên viết rule thành một README đầy đủ và khổng lồ. File quá dài sẽ khiến các rule quan trọng bị nhấn chìm, Codex cũng chưa chắc nghe lời hơn.

Tiêu chuẩn của tôi rất đơn giản:

> Nếu xóa dòng này, Codex có dễ mắc lỗi hơn không?

Có thì giữ; không thì xóa.

Một số team còn dùng `AGENTS.md` làm sổ ghi lỗi của Agent. Ví dụ Codex liên tục sửa sai test command trong một loại nhiệm vụ nào đó, động nhầm directory được generate, quên chạy một check nào đó, thì ghi nguyên nhân và cách làm đúng vào đó. Ý tưởng này đúng, nhưng đừng dán nguyên trạng mọi lần thất bại vào. Tốt nhất nén thành một rule có thể thực thi, nếu không file sẽ nhanh chóng biến thành nhật ký dài dòng.

`/init` có thể generate một `AGENTS.md` ban đầu, nhưng chỉ nên coi đó là bản nháp. Nội dung được generate tự động thường chép cả nội dung trong README vào, cũng có thể đoán sai test command. Sau khi generate, tốt nhất hãy tự xóa một lượt, chỉ giữ lại phần ảnh hưởng đến hành vi của Codex.

Có một cách viết khác phù hợp hơn với project lớn: để `AGENTS.md` chỉ làm mục lục.

Trong bài [Giải thích Harness Engineering trong một bài viết](https://javaguide.cn/ai/agent/harness-engineering.html), tôi cũng từng đề cập rằng `AGENTS.md` của chính OpenAI chỉ khoảng 100 dòng, giống một index hơn: trước tiên nói cho Agent các rule quan trọng nhất của repository, sau đó trỏ đến các tài liệu design chi tiết hơn, architecture diagram, execution plan và quality rating bên dưới `docs/`. Khi Agent thực sự cần đi sâu vào một module nào đó, nó sẽ đi theo link để đọc tiếp.

Đó chính là progressive disclosure.

Đừng nhồi toàn bộ bối cảnh vào context cùng một lúc. `AGENTS.md` ở root chỉ đặt các work rule quan trọng nhất; `AGENTS.md` cấp module đặt quy ước cục bộ; các tài liệu design dài hơn, bối cảnh migration và lựa chọn architecture đặt trong file riêng, rồi dùng link để Agent load khi cần. Cách này vừa không lãng phí context, vừa dễ bảo trì hơn.

## `config.toml` quản lý hành vi của client

`AGENTS.md` là hướng dẫn của project, còn `config.toml` là configuration của chính Codex client.

Một số vị trí thường gặp: configuration cấp user ở `~/.codex/config.toml`, configuration cấp project ở `.codex/config.toml`, các profile khác nhau có thể đặt ở `~/.codex/<profile>.config.toml`, configuration cấp system trên Unix thường ở `/etc/codex/config.toml`.

Theo tài liệu configuration chính thức hiện tại, priority từ cao xuống thấp là: CLI flags và override bằng `--config`, `.codex/config.toml` cấp project, profile được chọn qua `--profile`, `~/.codex/config.toml` cấp user, `/etc/codex/config.toml` cấp system, rồi đến default value tích hợp sẵn. Configuration cấp project chỉ được load sau khi project đã được trust; nếu project bị đánh dấu là untrusted thì configuration, Hooks và Rules trong `.codex/` của project đều bị bỏ qua.

Trong sử dụng hằng ngày, điều đáng quan tâm nhất không phải tên một model nào đó, mà là quyền hạn và sandbox.

```toml
approval_policy = "on-request"
sandbox_mode = "workspace-write"
```

Bộ configuration này khá phù hợp cho development hằng ngày: Codex có thể sửa file trong workspace và chạy verify, nhưng sẽ dừng lại hỏi bạn khi gặp command nhạy cảm hơn.

`approval_policy = "never"` hoặc sandbox rộng hơn không phải không thể dùng, chỉ là nên đặt trong môi trường đã được isolation. Ví dụ temporary worktree, container, script dùng một lần, CI, test account và credential với permission tối thiểu. Chỉ để giảm vài lần xác nhận mà mở toàn bộ quyền trong project thực tế thì không đáng.

Hooks hiện được bật theo mặc định. Nếu thực sự muốn tắt, hãy đặt trong `config.toml`:

```toml
[features]
hooks = false
```

Hướng này cũng phù hợp hơn với trực giác security: configuration, Rules và Hooks đi kèm repository của người khác đều có thể ảnh hưởng đến việc execute cục bộ, không thể mặc định tin tưởng tất cả.

Có thể tạm ghi nhớ vai trò của các file và cơ chế này như sau:

| Khả năng           | Chủ yếu giải quyết gì               | Phù hợp để đặt gì                                                           |
| ------------------ | ----------------------------------- | --------------------------------------------------------------------------- |
| `AGENTS.md`        | Hướng dẫn công việc cho Agent       | Rule project, command thường dùng, quy ước directory, tiêu chuẩn nghiệm thu |
| `config.toml`      | Configuration của Codex client      | Configuration model, sandbox, approval, profile, MCP                        |
| Rules              | allow / prompt / forbid cấp command | Command nào được phép, command nào phải xác nhận, command nào bị cấm        |
| Hooks              | Script theo lifecycle               | Check, audit, format, inject context                                        |
| sandbox / approval | Ranh giới execute cuối cùng         | File system, network, execute command và strategy xác nhận thủ công         |

## Mỗi thứ quản lý một phần: quyền hạn, Rules và Hooks

Codex có nhiều lớp security control, người mới bắt đầu rất dễ viết tất cả vào `AGENTS.md`.

Cách này không đủ đáng tin, vì `AGENTS.md` chỉ là instruction chứ không phải hard constraint ở tầng execute.

`AGENTS.md` là nhắc nhở mềm; sandbox và approval quản lý execution boundary; Rules quản lý command có được chạy hay không; Hooks quản lý việc bắt buộc phải làm tại một lifecycle node nào đó.

Ví dụ “đừng execute `rm -rf`”, nếu chỉ viết trong `AGENTS.md` thì vẫn là một đề xuất. Viết vào Rules thì Codex sẽ bị chặn trước khi execute. Rules hiện vẫn là khả năng experimental, syntax và mức độ hoàn thiện có thể thay đổi; cách viết dưới đây dựa trên tài liệu Codex Rules hiện tại. Nếu version trên máy bạn không hỗ trợ, trước tiên hãy dùng `/permissions`, sandbox, approval hoặc Hooks để thay thế việc control.

```python
prefix_rule(
    pattern = ["rm", "-rf"],
    decision = "forbidden",
    justification = "Không để Codex thực hiện xóa cưỡng chế đệ quy; hãy xác nhận thủ công directory cụ thể rồi tự xử lý.",
    match = [
        "rm -rf dist",
    ],
)
```

Hooks giải quyết một loại vấn đề khác.

Nếu muốn Codex chạy một script kiểm tra trước khi dừng, hoặc kiểm tra trước khi gọi tool xem prompt có vô tình dán API key hay không, hoặc tự động format sau khi edit, thì phù hợp để đặt vào Hook. Các event mà Codex Hooks hỗ trợ hiện tại hãy xem tài liệu chính thức, ví dụ `PreToolUse`, `PermissionRequest`, `PostToolUse`, `PreCompact`, `PostCompact`, `UserPromptSubmit`, `SessionStart`, `SubagentStart`, `SubagentStop`, `Stop` và các event khác.

Tuy nhiên Hook cuối cùng vẫn chạy local script, viết sai thì vẫn phiền. Tài liệu chính thức cũng đề cập rằng command Hook không được quản lý cần Review và trust, sau khi thay đổi sẽ lại chờ xác nhận. Nhiều command hook cùng match một event sẽ được khởi động song song, không thể dựa vào thứ tự execute giữa các Hook để chặn security. Giới hạn này trông có vẻ rườm rà, nhưng khá cần thiết.

## Để Codex chứng minh nó thực sự sửa đúng

Điểm phiền nhất khi AI viết code không phải là nó không viết được, mà là nó rất giỏi viết ra code “trông hợp lý”.

Vì vậy tôi hiếm khi chỉ nói “sửa xong thì báo tôi”. Tôi thích viết việc verify vào trong nhiệm vụ hơn:

```text
Trước tiên bổ sung một test thất bại để tái hiện vấn đề này.
Xác nhận test thất bại rồi mới sửa implementation.
Sau khi sửa hãy chạy các test liên quan.
Nếu vẫn thất bại liên tiếp hai hoặc ba lượt, hãy dừng lại và báo cáo điểm đang block cùng bằng chứng, đừng tiếp tục sửa mù quáng.
```

Thứ tự này có thể chặn rất nhiều fake fix. Nó phải tái hiện vấn đề trước, sau đó sửa implementation, cuối cùng dùng test để chứng minh.

Khi Codex kết thúc, tôi thường xem 3 việc: đã sửa những file nào, đã chạy những command nào, còn risk nào chưa được cover. `/diff` dùng để xem nhanh thay đổi, `/review` có thể review thay đổi chưa commit hiện tại, một commit nào đó hoặc thực hiện check theo yêu cầu tùy chỉnh của bạn.

Chi tiết hơn, có thể yêu cầu bằng chứng verify của AI Coding theo checklist này:

- Test thất bại trước, sau đó chuyển sang pass.
- Tóm tắt `git diff` và mô tả các file quan trọng.
- Command test, lint, build và kết quả.
- Các risk chưa được cover.
- Trọng điểm cần human Review.
- Cách rollback khi xảy ra vấn đề.

Trong thực tiễn cộng đồng có hai prompt khá hữu ích:

```text
Prove to me this works. Compare the diff against main and show the evidence.
```

```text
Knowing everything you know now, scrap this approach and propose the simpler implementation.
```

Câu trước yêu cầu nó đưa ra bằng chứng, không chỉ viết kết luận. Câu sau phù hợp khi phương án phiên bản đầu chạy được nhưng rất rối. Codex đã đọc một lượt context, để nó suy nghĩ lại một lần nữa thường có thể làm implementation gọn gàng hơn.

Tuy nhiên cuối cùng vẫn phải tự xem diff. Summary của Codex không thể thay thế Review. Nó nói “chỉ sửa test”, bạn vẫn phải mở file quan trọng xem qua; nó nói “không có risk”, bạn cũng phải tự suy nghĩ xem transaction, concurrency, permission và compatibility có bị bỏ sót không.

## MCP chỉ kết nối những tool thực sự giúp tiết kiệm công sức

MCP (Model Context Protocol, giao thức context của model) giống một quy chuẩn kết nối: **hệ thống bên ngoài đóng gói capability thành MCP Server, sau khi ứng dụng AI hỗ trợ MCP kết nối vào thì có thể phát hiện và gọi những capability này.**

![Sơ đồ MCP](https://oss.javaguide.cn/github/javaguide/ai/skills/mcp-simple-diagram.png)

Context trong development thực tế không chỉ nằm trong repository.

Error ở Sentry, requirement ở Linear, API documentation ở tài liệu nội bộ, design ở Figma, các bước reproduce ở browser, thảo luận PR ở GitHub. Bạn đương nhiên có thể copy từng đoạn cho Codex, nhưng làm nhiều lần sẽ rất phiền.

MCP phù hợp để giải quyết loại vấn đề này. Theo tài liệu MCP hiện tại của Codex, Codex hỗ trợ STDIO MCP Server và Streamable HTTP Server; Streamable HTTP Server hỗ trợ authentication bằng Bearer token hoặc OAuth. Loại server cụ thể, authentication field và cách configuration vẫn phải căn cứ vào tài liệu MCP hiện tại.

Ví dụ thêm Context7 documentation MCP:

```bash
codex mcp add context7 -- npx -y @upstash/context7-mcp
```

Sau khi thêm, có thể dùng `/mcp` trong TUI để xem trạng thái server hiện tại.

Ở đây có một điểm cần cân nhắc: **MCP không phải càng nhiều càng tốt.**

Tôi khuyên chỉ kết nối các tool có tần suất cao, mục đích rõ ràng và tốt nhất ban đầu chỉ read-only. Nếu thường xuyên tra lỗi production thì kết nối Sentry hoặc nền tảng log; thường xuyên sửa frontend thì kết nối browser, Playwright, Figma; thường xuyên xử lý PR thì kết nối GitHub. Với MCP có write permission, có token và có thể thao tác hệ thống bên ngoài, trước tiên nên tiết chế.

Có thể chia thành ba lớp theo risk:

- MCP read-only: tra documentation, tra error log, đọc Sentry, xem thông tin PR.
- MCP half-write: tạo issue, comment PR, tạo draft, cập nhật documentation không phải production.
- MCP high-risk: release, sửa configuration production, xóa resource, thao tác cloud platform hoặc database.

Mặc định trước tiên hãy kết nối tool read-only. Tool half-write cần giới hạn scope, tool high-risk cần approval và audit riêng, token cố gắng dùng permission tối thiểu và credential ngắn hạn.

Càng nhiều tool, không gian lựa chọn của Codex càng lớn, xác suất dùng sai cũng sẽ tăng.

Khi tự viết MCP Server, đừng chỉ expose tool parameter. Tài liệu MCP hiện tại của Codex đề cập rằng Codex sẽ đọc field `instructions` được trả về khi MCP khởi tạo, đồng thời khuyên đặt hướng dẫn quan trọng nhất trong 512 ký tự đầu tiên. Khi nào nên dùng, khi nào không nên dùng, cần hiểu nội dung trả về thế nào, những điều này đều đáng viết rõ.

## Dùng Skills để lưu workflow lặp lại

Rule file và Skill giải quyết các vấn đề hơi khác nhau.

Rule file phù hợp hơn để đặt những điều project này luôn phải tuân thủ, ví dụ: version tech stack, command khởi động, cấu trúc directory, format error code và những file không được đụng vào.

Skill phù hợp hơn để đặt cách thực hiện khi gặp một loại nhiệm vụ nào đó. Ví dụ code review, viết test, sửa frontend page, research web hay viết technical article; nếu workflow của các nhiệm vụ này gần như giống nhau mỗi lần, thì không cần lần nào cũng nhắc lại trong chat.

Trước đây G từng viết hai bài liên quan: [Agent Skills là gì? Khác Prompt và MCP chính xác ở đâu?](https://javaguide.cn/ai/agent/skills.html) và [Danh sách lựa chọn Skills thiết yếu cho AI Coding](https://javaguide.cn/ai-coding/practices/programmer-essential-skills.html).

Nói đơn giản, Skill là một hướng dẫn nhiệm vụ có thể được Agent load khi cần. Nó không phải plugin, cũng không phải bản thân MCP tool, mà là việc viết workflow, constraint, mục kiểm tra và kinh nghiệm tránh lỗi của một loại nhiệm vụ vào `SKILL.md`.

Skill không giống `AGENTS.md` ở chỗ mỗi lần đều nhồi toàn bộ nội dung vào context. Theo mặc định, Codex trước tiên sẽ thấy tên và description của Skill để phán đoán có nên gọi hay không; chỉ khi thực sự dùng Skill thì phần nội dung của `SKILL.md` và resource liên quan mới được đưa vào context.

![Execution flow của Agent](https://oss.javaguide.cn/github/javaguide/ai/skills/skills-agent-execution-link.png)

Những workflow có tính lặp lại cao này đều phù hợp để tích lũy thành Skill. Ví dụ trước khi viết feature luôn cố định đi qua TDD, viết test thất bại trước rồi mới implementation; khi code review luôn cố định kiểm tra security, transaction, performance và edge case; khi viết technical article luôn cố định kiểm tra fact source, citation, cấp heading và dấu vết AI.

Giá trị của Skill nằm ở đây: biến việc nhắc lại thành sổ tay công việc có thể tái sử dụng. Skills của Codex và Skills của Claude khá gần nhau về ý tưởng, đều tích lũy workflow của nhiệm vụ lặp lại thành capability có thể dùng lại; nhưng cấu trúc file, cách trigger, platform có thể dùng và security model của hai bên phải lần lượt căn cứ vào documentation chính thức tương ứng.

Một `SKILL.md` tối thiểu có thể dùng được trước tiên có thể viết ở mức độ này:

```markdown
---
name: java-service-review
description: Review Java service-layer changes for transaction boundaries, null handling, logging, and regression tests.
---

Use this skill when reviewing Java service-layer changes.

Input materials:

- Current diff or target files.
- Related tests and error logs, if available.

Steps:

1. Read the changed service methods and related tests.
2. Check transaction boundaries, null handling, logging, and regression coverage.
3. Return findings with file and line references.

Do not rewrite code unless the user explicitly asks.

Done when:

- Findings are ordered by severity.
- Each finding explains the risk and a concrete fix direction.
```

Skill có sẵn cũng có thể dùng trực tiếp, ví dụ Superpowers đã đóng gói các workflow TDD, Code Review, Spec-Driven, Git Worktree và phối hợp sub-agent.

Trong bài [Danh sách lựa chọn Skills cho AI Coding: làm rõ requirement, TDD, code review và UI design](https://javaguide.cn/ai-coding/practices/programmer-essential-skills.html), tôi có đề xuất chi tiết.

Nhưng đừng lấy Skill bên thứ ba rồi chạy ngay. `SKILL.md` cũng là instruction; nếu bên trong có command nguy hiểm, script kỳ lạ hoặc permission quá rộng, Agent sẽ làm theo. Trước khi cài đặt ít nhất hãy xem qua nội dung, `scripts/` và `references/`, xác nhận nó không thực hiện thao tác vượt quyền.

## Subagents phù hợp để khảo sát nhánh phụ

Trong nhiệm vụ dài, phần chiếm context nhiều nhất thường không phải phương án cuối cùng mà là quá trình khảo sát ở giữa.

Ví dụ điều tra một Bug phức tạp, Codex có thể phải đọc hàng chục file, xem một đống log và thử vài giả thuyết. Cuối cùng kết luận thực sự hữu ích chỉ có vài ý, nhưng main session đã bị lấp đầy bởi search result và suy luận trung gian.

Lúc này có thể dùng Subagents.

Theo documentation Codex hiện tại, Subagent workflow được bật theo mặc định, nhưng Codex chỉ spawn subagents khi bạn yêu cầu rõ ràng. Mỗi subagent đều thực hiện model work và tool work của riêng mình, vì vậy sẽ tốn token hơn một agent.

Các agent tích hợp được liệt kê trong documentation hiện tại gồm: `default` làm fallback dùng chung, `worker` thiên về execute và fix, `explorer` thiên về read-only exploration. Format configuration của custom agent, tên agent tích hợp và entry point có thể thay đổi theo version; thực tế hãy căn cứ vào `/agent`, documentation Subagents chính thức và version trên máy. Bạn cũng có thể đặt custom TOML Agent trong `~/.codex/agents/` hoặc `.codex/agents/`.

Những nhiệm vụ khá phù hợp để tách ra thường có dạng:

```text
Review current branch against main.
Spawn one subagent for each topic: security, concurrency, tests, maintainability.
Wait for all agents, then summarize findings with file references and severity.
Do not modify files.
```

Loại nhiệm vụ này có boundary rõ ràng và tự nhiên có thể chạy song song.

Không phù hợp để tách là thay đổi rất nhỏ. Sửa một field của DTO mà mở 4 subagent thì communication cost có thể còn cao hơn chính việc sửa. Thói quen của tôi là: main session phụ trách mục tiêu, trade-off và nghiệm thu cuối cùng; subagent chỉ xử lý phần cục bộ, rõ ràng và có thể báo cáo độc lập.

Ngoài ra cần lưu ý: Subagents kế thừa sandbox policy hiện tại. Trong CLI interactive, approval request của thread không phải thread hiện tại cũng có thể hiện ra; trước khi approve hãy xem rõ request do agent nào khởi tạo.

## Đừng tự động hóa toàn bộ Scheduled Tasks ngay từ đầu

Khả năng trước đây thường được gọi là Automations trong Codex App, hiện UI chủ yếu gọi là **Scheduled Tasks**. Nó phù hợp để chạy nhiệm vụ lặp lại, ví dụ quét commit gần đây mỗi ngày, generate release note mỗi tuần, định kỳ kiểm tra CI thất bại hoặc tổng hợp alert chưa xử lý.

Nó không phải thứ dùng để “tự động fix mọi thứ”.

Scheduled Tasks cần phân biệt cách chạy. Schedule gắn với task hiện tại phù hợp để quay lại cùng context và tiếp tục kiểm tra; schedule độc lập hoặc cấp project có thể khởi động theo schedule. Khi task cấp project execute, máy đang chạy Codex App cục bộ phải bật, Codex phải đang chạy và project path cũng phải còn trên disk. Task của Git repository có thể chạy trong project cục bộ hoặc trong dedicated background worktree. Scheduled Tasks dùng sandbox setting mặc định; nếu cấp full access thì risk của background task cũng tăng.

Trình tự tôi thấy ổn hơn là: trước tiên viết workflow thành prompt thông thường, chạy thủ công vài lần; nếu lần nào cũng copy cùng một bộ bước thì tích lũy thành Skill; sau khi Skill ổn định rồi mới tạo thành Scheduled Task.

Nói cách khác, Skill định nghĩa method, Scheduled Task định nghĩa time. Prompt của task cũng phải viết thành durable prompt có thể chạy độc lập, đừng phụ thuộc vào context ngầm của cuộc trò chuyện trước.

Ví dụ “mỗi ngày tự động fix mọi Bug rồi tạo PR”, nghe có vẻ rất tiện, nhưng trong project thực tế rất có thể tạo ra một đống diff cần con người dọn dẹp. Đáng tin cậy hơn là “mỗi ngày scan các CI failure trong 24 giờ gần nhất và tổng hợp nguyên nhân”. Trước tiên để nó report, sau đó mới quyết định có sửa hay không.

## Chỉ cần nhớ vài nhóm command thường dùng

Slash command của Codex CLI sẽ thay đổi; các command thấy được trong CLI, Codex App và IDE Extension cũng không nhất định hoàn toàn giống nhau. Những command dưới đây chỉ là kinh nghiệm sử dụng hiện tại, thực tế hãy căn cứ vào popup `/` và `/help` trên surface bạn đang dùng.

Thông thường tôi chỉ nhớ vài nhóm:

- Điều khiển session và kế hoạch: `/permissions`, `/model`, `/fast`, `/status`, `/clear`, `/plan`, `/goal`.
- Xem context, memory và thay đổi: `/diff`, `/compact`, `/copy`, `/memories`.
- Mở rộng capability: `/agent`, `/mcp`, `/hooks`, `/plugins`, `/apps`, `/skills`.
- Review và khôi phục: `/review`, `/fork`, `/resume`.

Command chỉ là entry point, không phải bản thân workflow. Thứ thực sự quyết định kết quả vẫn là boundary của task, rule của project, tiêu chuẩn verify và configuration quyền hạn.

## Một số workflow tôi thường dùng

Khi tiếp nhận project chưa quen, trước tiên tôi sẽ để Codex làm hướng dẫn viên tạm thời:

```text
Không sửa file.
Hãy giải thích flow đăng nhập của người dùng, từ lúc HTTP request đi vào cho đến khi session được ghi.
Liệt kê các class, method, configuration quan trọng và các implicit constraint mà bạn cho rằng con người cần xác nhận.
```

Cần spot check nội dung nó tổng hợp, đặc biệt là cross-service call, gray configuration và logic compatibility lịch sử. Để nó liệt kê file và method name sẽ đáng tin cậy hơn chỉ nghe summary bằng ngôn ngữ tự nhiên.

Khi fix Bug, đừng chỉ nói “hãy sửa giúp tôi”. Tôi thích bày toàn bộ tài liệu ra hơn:

```text
Dưới đây là test thất bại, error log và các bước reproduce.
Trước tiên hãy định vị root cause, đừng lập tức sửa code.
Sau khi tìm được root cause, trước tiên bổ sung một test có thể tái hiện, rồi mới sửa implementation.
Sau khi hoàn thành hãy chạy các test liên quan và giải thích vì sao test này có thể cover vấn đề.
```

Nếu nó liên tục xoay quanh cùng một hướng sai trong hai lượt, đừng tiếp tục hỏi “thử lại đi”. Hãy dừng lại và để nó tổng kết những gì đã biết, giả thuyết nào đã bị bác bỏ và bước tiếp theo còn thiếu bằng chứng gì.

TDD cũng rất hữu ích với AI Coding:

```text
Trước tiên chưa sửa implementation.
Viết một test thất bại cho OrderStatusService, cover trường hợp callback lặp lại của order đã thanh toán không được trừ inventory lặp lại.
Sau khi test thất bại thì sửa implementation cho đến khi test pass.
```

Trình tự này giúp cố định behavior mong đợi trước, sau đó để Codex implementation.

Với task frontend cần cụ thể hơn một chút. Đừng chỉ nói “hiện đại, đơn giản, cao cấp”, những từ này quá trống rỗng, cuối cùng rất dễ nhận được gradient tím, card bo góc lớn và layout kiểu marketing page. Đặc biệt backend system rất dễ thất bại theo cách này.

```text
Đây là trang vận hành backend, ưu tiên mật độ thông tin, không dùng phong cách marketing page.
Dùng component Ant Design hiện tại, không thêm UI library.
Tham khảo khu vực filter, table và layout pagination của src/pages/UserList.tsx.
Giữ màu chủ đạo theo CSS variable, không thêm gradient background.
Sau khi hoàn thành hãy khởi động page local, kiểm tra trên mobile và desktop xem có text bị chồng lên nhau không.
```

PR Review cũng vậy, phạm vi càng hẹp càng tốt:

```text
Review current branch against main.
Focus only on correctness, transaction boundaries, null handling, and missing tests.
Return findings ordered by severity with file and line references.
Do not comment on style unless it can cause a bug.
```

Đôi khi Codex nói “có thể tốt hơn” như thể “bắt buộc phải sửa”. Trong kết quả Review, thứ thực sự cần ưu tiên xử lý là các finding có thể gây Bug, vấn đề security, data inconsistency, phá vỡ compatibility và thiếu test.

## Ranh giới security

Codex có thể đọc file, ghi file, chạy command, kết nối MCP và điều khiển browser. Capability càng mạnh thì boundary càng phải rõ.

Tôi đề xuất ít nhất phải giữ vững một số ranh giới:

- Không expose password database production, token dài hạn của cloud provider và SSH key cho Codex.
- Không để nó mặc định đọc `.env`, certificate, database dump, production log và directory private key.
- Không để nó trực tiếp thao tác production environment, trừ khi có credential tạm thời, approval và audit.
- Không mặc định cho phép push vào main branch hoặc force-push remote branch.
- Không execute remote script không rõ nguồn trong environment không isolation.
- Không kết nối toàn bộ MCP có write permission cùng một lúc.

Nếu thực sự cần chạy automation quyền cao, hãy đặt nó trong container, dùng account tạm thời, credential permission tối thiểu và worktree độc lập. AI viết sai code còn có thể Review, AI cầm sai permission thì phiền hơn nhiều. Automation quyền cao cũng cần giữ operation log, command output, diff và approval record, đồng thời bảo đảm có thể rollback nhanh.

## Những điểm dễ gặp sự cố

Task quá mơ hồ là vấn đề thường gặp nhất. Bạn chỉ nói “tối ưu một chút”, Codex chỉ có thể tự đoán, cuối cùng rất có thể search một đống file rồi sửa một đống code ở rìa. Bổ sung đầy đủ mục tiêu, context, constraint và tiêu chuẩn hoàn thành thường có thể giảm rất nhiều exploration vô ích.

Lập kế hoạch quá mức cũng lãng phí thời gian. Thay đổi nhỏ không cần plan dài, cứ làm trực tiếp, xem diff rồi chạy verify là được. Giai đoạn lập kế hoạch (Plan) phù hợp hơn với nhiệm vụ cross-module, risk cao và call chain không rõ.

Khi `AGENTS.md` quá dài, hiệu quả ngược lại có thể giảm. Có rất nhiều rule, nhưng vài rule thực sự quan trọng lại bị làm loãng. Nó nên phát triển từ lỗi thực tế: những vấn đề Codex liên tục mắc phải thì ghi vào; sự thật có thể đọc ra ngay từ code thì xóa đi.

Tool và permission cũng đừng mở quá rộng ngay một lần. Kết nối quá nhiều MCP khiến Codex chọn sai; cấp permission quá rộng khiến background task có thể làm những việc vượt quá dự đoán của bạn. Task quyền cao nên đặt trong environment isolation, development hằng ngày giữ permission tối thiểu.

Cuối cùng là thiếu verify. Code trông hợp lý không có nghĩa behavior đúng. Test, lint, build, screenshot và log, ít nhất phải có một loại. Khi session dài bắt đầu chậm và thiếu ổn định, hãy dùng `/compact`, khi cần thì `/fork` hoặc mở thread mới. Multi-agent cũng vậy, main session đưa ra quyết định, subagent chỉ xử lý phần research cục bộ.

## Sử dụng theo phân tầng risk

Task nhỏ không cần phức tạp hóa. Sửa text, bổ sung log, sửa một field mapping rõ ràng thì cứ để Codex execute trực tiếp, sau đó xem `/diff` và chạy unit test tương ứng là được.

Task trung bình nên đi qua giai đoạn lập kế hoạch (Plan), sau đó execute rồi verify. Ví dụ sửa business flow trong một module, bổ sung một API hoặc refactor một service cục bộ, tốt nhất trước tiên để Codex đọc các file liên quan, liệt kê điểm sửa và command verify, sau khi bạn xác nhận mới bắt tay vào làm.

Task risk cao trước tiên chỉ nên read-only analysis. Với thay đổi về payment, permission, data migration, configuration production và concurrency consistency, trước tiên để Codex tìm call chain, risk và test gap; sau khi con người xác nhận các phán đoán quan trọng, mới dùng TDD hoặc commit nhỏ từng bước để triển khai. Về environment, cố gắng dùng worktree, container, credential tạm thời và permission chặt hơn.

Automation task cũng đừng làm một bước là xong. Trước tiên chạy thủ công một hoặc hai lần, sau đó tích lũy thành Skill; khi Skill ổn định rồi mới tạo thành Scheduled Task. Automation quyền cao cần giữ thêm audit record và phương án rollback.

## Tổng kết

Sau khi dùng Codex thành thạo, cảm giác sẽ chuyển từ “để AI viết code” thành “điều phối một trợ lý kỹ thuật có thể tự đọc repository, chạy command và bàn giao diff”.

Nhưng càng như vậy thì càng không thể chỉ nhìn vào prompt.

Boundary của task, rule của project, kiểm soát permission, tiêu chuẩn verify, tool bên ngoài và workflow có thể tái sử dụng, tất cả cùng quyết định chất lượng cuối cùng Codex bàn giao. Đề xuất của tôi vẫn là câu đó: trước tiên để nó làm đúng và ổn định trong phạm vi nhỏ, sau đó từ từ mở rộng boundary ra ngoài.

Đừng tự động hóa toàn bộ ngay từ đầu.
