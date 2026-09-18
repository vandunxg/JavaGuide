---
title: "Danh sách lựa chọn AI Coding Skills: Làm rõ yêu cầu, TDD, code review và thiết kế UI"
description: Danh sách lựa chọn AI Coding Skills được sắp xếp theo bối cảnh tác vụ, bao quát làm rõ yêu cầu, TDD, code review, thiết kế UI, tối ưu performance React, PostgreSQL, Claude API và phát triển Skill, đồng thời nêu rõ công cụ nào phù hợp để cài đặt theo nhu cầu.
category: AI Coding thực chiến
head:
  - - meta
    - name: keywords
      content: AI Coding,Skills,Superpowers,mattpocock,grilling,Claude Code,Cursor,code review,TDD,thiết kế UI,React,Next.js,PostgreSQL,Claude API,phát triển Skill
---

Xin chào, mình là G nhỏ. Gần đây có người bạn hỏi mình: “Bình thường bạn hay dùng những Skill nào? Có thể chia sẻ cho mọi người một chút không?”

Tất nhiên là được! Skill vừa xuất hiện thì mình đã bắt đầu dùng, sau đó cũng từng tùy chỉnh khá nhiều Skill trong công ty. Đồng nghiệp và bạn bè dùng đều nói là tốt.

Theo lý mà nói, lập một danh sách hẳn khá đơn giản. Nhưng khi thật sự bắt tay sắp xếp, mình phát hiện Skill đủ điều kiện vào danh sách thực ra không nhiều. Hơn nữa, trong quá trình sắp xếp Skill, mình lại phải cảm thán rằng công nghệ thời AI phát triển quá nhanh! Hơi không theo kịp rồi!

Khi Skill mới xuất hiện, năng lực của model vẫn chưa mạnh như vậy. Chưa đọc hiểu dự án đã bắt tay làm, viết code xong không bổ sung test, trò chuyện lâu rồi quên mất yêu cầu phía trước, những tình huống này rất thường gặp.

Vì vậy, chúng ta nhồi từng bước phát triển vào Skill: trước hết làm rõ yêu cầu, sau đó tách plan, phải viết test trước, sửa xong còn phải review. Quy tắc càng chi tiết thì càng yên tâm.

Nhưng model mạnh lên quá nhanh.

Hiện nay, khi giao cùng một task cho Codex hoặc Claude Code, phần lớn thời gian chúng sẽ tự đọc project, tra call chain, sửa file, bổ sung test và chạy verification. Những việc trước đây cần liên tục nhắc trong `SKILL.md` giờ đã trở thành thao tác cơ bản.

Lúc này mà cài đầy một lượt các Skills cũng giống như nhét cả chồng sổ tay hướng dẫn cho một người đã biết làm việc. Một thay đổi nhỏ cũng phải viết plan, qua review, chạy verification đầy đủ; cuối cùng toàn bộ thời gian đều tiêu tốn vào quy trình, hơi có phần rườm rà.

Vì vậy danh sách này sẽ được giữ khá chừng mực. Những gì hiện giờ mình muốn giữ lại thường có thể bổ sung quy ước project mà model không đoán được, hoặc có sẵn quy trình chuyên môn, script, template và tài liệu tham khảo. Những Skill chỉ nhắc “đọc code trước, rồi sửa, cuối cùng chạy test” thì mình cơ bản không còn đề xuất nữa.

Bài [giải thích chi tiết Agent Skills dài vạn chữ](https://javaguide.cn/ai/agent/skills.html) trước đây đã nói về khác biệt giữa Skill với Prompt và MCP; nếu muốn biết vì sao mình bắt đầu cắt giảm Skills, bạn có thể đọc tiếp bài mới nhất [Trong thời đại model mạnh, AI Coding Skills còn cần cài không?](./skill-selection-and-pruning.md).

## Superpowers

Superpowers kết nối làm rõ yêu cầu, plan, TDD, Git Worktree, cộng tác với sub Agent, code review và verification trước khi hoàn tất thành một phương pháp phát triển hoàn chỉnh. Nó phù hợp với codebase xa lạ, feature phức tạp và thay đổi rủi ro cao; khi sửa nội dung, bổ sung kiểm tra giá trị rỗng hoặc điều chỉnh một logic validation, quy trình này thường quá nặng.

Superpowers lần lượt gọi các Skill sau:

| Skill                                             | Thao tác chính                                                       |
| ------------------------------------------------- | -------------------------------------------------------------------- |
| `brainstorming`                                   | Làm rõ yêu cầu, so sánh phương án và lưu thiết kế trước khi coding   |
| `using-git-worktrees`                             | Tạo workspace cô lập, kiểm tra test baseline                         |
| `writing-plans`                                   | Tách thiết kế thành các task nhỏ có vị trí file và bước verification |
| `subagent-driven-development` / `executing-plans` | Phân công sub Agent hoặc thực thi plan theo batch                    |
| `test-driven-development`                         | Thực hiện RED-GREEN-REFACTOR                                         |
| `requesting-code-review`                          | Review implementation theo mức độ nghiêm trọng                       |
| `verification-before-completion`                  | Kiểm tra bằng chứng trước khi thông báo hoàn tất                     |
| `finishing-a-development-branch`                  | Verify test và xử lý merge, PR hoặc giữ branch                       |

Quy trình này đã tương thích với Claude Code, Codex, Cursor và OpenCode. Lệnh cài plugin tương ứng của Claude Code là:

```text
/plugin install superpowers@claude-plugins-official
```

Codex App có thể tìm Superpowers trong Plugins ở sidebar; Codex CLI có thể nhập `/plugins` rồi tìm để cài đặt. Cách cài đặt của các Agent khác nhau không dùng chung được; nếu sử dụng nhiều Agent thì cần cài riêng cho từng Agent.

Giao diện cài đặt Claude Code sẽ yêu cầu bạn chọn phạm vi áp dụng:

![Tải xuống Superpowers](https://oss.javaguide.cn/github/javaguide/ai/superpowers/superpowers-download.png)

| Tùy chọn                           | Phạm vi áp dụng                     | Đề xuất                                                                                     |
| ---------------------------------- | ----------------------------------- | ------------------------------------------------------------------------------------------- |
| Install for you                    | Có hiệu lực với mọi project         | Chỉ chọn khi đã xác nhận muốn dùng lâu dài toàn bộ quy trình                                |
| Install for all collaborators      | Chia sẻ với thành viên project      | Chỉ commit khi team đã thống nhất dùng cùng một phương pháp phát triển                      |
| Install for you, in this repo only | Chỉ có hiệu lực trong repo hiện tại | Ưu tiên chọn khi trải nghiệm lần đầu, thuận tiện quan sát nó có làm chậm task nhỏ hay không |

Mình không còn đề xuất người mới cài global ngay từ đầu. Hãy chạy hai, ba task thực tế trong một repo trước, xem quy trình làm rõ yêu cầu, TDD và review có thật sự giảm việc làm lại hay không, rồi mới quyết định có mở rộng phạm vi không.

Địa chỉ project: <https://github.com/obra/superpowers>

## mattpocock/skills

Nếu chỉ muốn tăng cường một khâu trong quy trình phát triển, bạn có thể xem [mattpocock/skills](https://github.com/mattpocock/skills). Project này tách kinh nghiệm engineering thành các Skills nhỏ, dễ sửa và có thể kết hợp, không yêu cầu mọi task đều đi qua cùng một quy trình.

Các module hiện khá phù hợp với programmer gồm:

| Skill                    | Vấn đề phù hợp để giải quyết                                                                                |
| ------------------------ | ----------------------------------------------------------------------------------------------------------- |
| `grill-me` / `grilling`  | Liên tục đặt câu hỏi trước khi bắt đầu, xác nhận rõ yêu cầu, quyết định và quan hệ phụ thuộc                |
| `diagnosing-bugs`        | Điều tra theo thứ tự reproduce, thu hẹp phạm vi, đưa ra giả thuyết, instrumentation, sửa và regression test |
| `tdd`                    | Dùng vòng lặp RED-GREEN-REFACTOR để phát triển feature hoặc sửa Bug                                         |
| `code-review`            | Kiểm tra riêng coding convention và implementation có phù hợp yêu cầu ban đầu không                         |
| `to-spec` / `to-tickets` | Sắp xếp thảo luận đã có thành specification, sau đó tách thành các task có quan hệ phụ thuộc                |
| `domain-modeling`        | Thống nhất thuật ngữ domain và ghi các quyết định quan trọng vào `CONTEXT.md` cùng ADR                      |

Có thể cài đặt cross-Agent bằng:

```bash
npx skills@latest add mattpocock/skills
```

Installer sẽ cho bạn chọn Skills cần dùng và Agent đích. Theo hướng dẫn cài đặt hiện tại của project, bạn còn phải chọn `/setup-matt-pocock-skills`, sau đó chạy một lần trong repo đích để hoàn tất cấu hình vị trí lưu Issue Tracker và tài liệu.

Khi dùng lần đầu, không cần cài toàn bộ. Nếu yêu cầu thường chưa được trao đổi rõ, trước tiên có thể chọn `grill-me` và `grilling`; nếu thường gặp Bug khó định vị thì bổ sung `diagnosing-bugs`.

Bộ Skills này phù hợp với người đã có thói quen phát triển cơ bản và chỉ muốn bổ sung một vài khâu yếu. Nếu project đã có template yêu cầu, quy chuẩn TDD và quy trình code review ổn định, cài lặp lại Skill tương ứng sẽ không giúp được nhiều.

Cách dùng thực tế và ranh giới áp dụng của các Skill này được mình viết riêng trong bài: [mattpocock/skills: 4 AI Coding Skill mình đề xuất nhất](https://javaguide.cn/ai-coding/practices/mattpocock-skills.html).

Địa chỉ project: <https://github.com/mattpocock/skills>

## ECC (trước đây là Everything Claude Code)

Everything Claude Code hiện đã đổi tên thành **ECC**.

Project đã mở rộng từ một bộ cấu hình Claude Code thành hệ thống Harness cross-Agent, bao quát Codex, Claude Code, Cursor, OpenCode và các công cụ khác.

Repo ECC đồng thời chứa Skills, Agents, Hooks, Rules, cùng quản lý memory, security scan, continuous learning và quy tắc engineering đa ngôn ngữ. Skills trong repo đã lên tới hàng trăm, giống một thư viện cấu hình Harness cấp team hơn. Khi team cần thống nhất cách Agent làm việc, chiến lược memory và security check, quản lý tập trung sẽ giảm không ít cấu hình trùng lặp.

![Context rot](https://oss.javaguide.cn/github/javaguide/ai/harness/context-rot-diagram.png)

Khi component tăng lên, chi phí lựa chọn cũng tăng theo. Nếu project chỉ thiếu code review hoặc TDD, có thể dùng selective installation của ECC, chỉ lấy những component tương ứng như Java code review, context persistence hoặc security scan, không cần nhét toàn bộ hệ thống vào từng repo.

Địa chỉ project: <https://github.com/affaan-m/ECC>

## Doc Co-Authoring

Trước khi programmer viết code, bước dễ bị đánh giá thấp nhất thực ra là: nói rõ yêu cầu.

Khi yêu cầu chưa rõ, AI Coding Agent sẽ rất nỗ lực tiến lên, nhưng hướng tiến lên chưa chắc đúng. Nó có thể trực tiếp biến một ý tưởng chưa xác định ranh giới thành implementation; cuối cùng code, test và tài liệu đều đầy đủ, chỉ là khác thực tế một khoảng.

**doc-coauthoring** trong repo Skills chính thức của Anthropic được chuẩn bị cho những tình huống như vậy. Trọng tâm của nó rất cụ thể: tách việc viết PRD, technical proposal, decision document và RFC thành một quy trình cộng tác; trước tiên xử lý context, structure và cách reader hiểu, còn trau chuốt câu chữ chỉ là việc phía sau.

doc-coauthoring xử lý một tài liệu qua ba giai đoạn:

| Giai đoạn                  | Làm gì                                                                                                                   |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| **Context Gathering**      | Trước tiên thu thập background, constraint, thảo luận trước đây, dependency kiến trúc và mối quan tâm của stakeholder    |
| **Refinement & Structure** | Iteration theo chương, trước tiên đặt câu hỏi và mở rộng, sau đó sàng lọc nội dung, cuối cùng viết thành các đoạn dễ đọc |
| **Reader Testing**         | Dùng Claude với context hoàn toàn mới để test tài liệu, kiểm tra reader có hiểu sai hoặc bỏ sót không                    |

Lấy module hoàn tiền đơn hàng làm ví dụ. Trước khi Coding Agent bắt đầu, có thể dùng doc-coauthoring sắp xếp một technical proposal ngắn, viết rõ từng mục về state machine hoàn tiền, idempotent của API, rollback inventory và coupon, cùng bù trừ thủ công sau khi thất bại.

Sau khi tài liệu được chốt, task nhỏ có thể trực tiếp giao technical proposal và acceptance criteria cho Agent; khi thay đổi liên quan đến nhiều module, tiếp tục dùng Superpowers để tách plan, viết test và review code. Mục đích chính là giải quyết bất đồng ở giai đoạn tài liệu, tránh phải thay đổi scope sau khi implementation hoàn tất.

Lệnh cài đặt Claude Code như sau:

```bash
/plugin marketplace add anthropics/skills
/plugin install example-skills@anthropic-agent-skills
```

`example-skills` là một nhóm Skill mẫu, doc-coauthoring chỉ là một trong số đó. Sau khi cài nhóm plugin này, skill-creator được nhắc đến phía sau cũng sẽ được cung cấp, không cần cài lặp lại.

Địa chỉ project: <https://github.com/anthropics/skills/tree/main/skills/doc-coauthoring>

## UI UX Pro Max

Đây là một Skill thiết kế UI/UX chuyên nghiệp dành riêng cho AI Coding Agent như Claude Code, Cursor và Windsurf.

![UI UX Pro Max](https://oss.javaguide.cn/github/javaguide/ai/harness/ui-ux-pro-max-skill.png)

Nó sẽ dựa vào loại sản phẩm và đặc điểm ngành để tạo design system, sau đó giao màu sắc, font, layout, animation và anti-pattern cho Agent thực thi. So với Skill nhẹ chỉ có vài đoạn prompt về thẩm mỹ, nó đi kèm một bộ tài liệu thiết kế có thể tra cứu.

Dữ liệu chính do phiên bản public hiện tại cung cấp gồm:

| Loại tài nguyên                | Số lượng    | Mô tả                                                                   |
| ------------------------------ | ----------- | ----------------------------------------------------------------------- |
| UI style                       | 84 loại     | Glassmorphism, Neumorphism, Bento Grid, AI-Native UI...                 |
| Loại sản phẩm và color palette | 192 nhóm    | Ghép theo các bối cảnh sản phẩm như SaaS, tài chính, y tế và e-commerce |
| Font pairing                   | 74 nhóm     | Bao gồm các tổ hợp Google Fonts                                         |
| Loại biểu đồ                   | 25 loại     | Hướng đến dashboard và trang phân tích                                  |
| Quy tắc suy luận               | 161 quy tắc | Tạo design system theo ngành                                            |
| Nguyên tắc UX                  | 98 quy tắc  | Bao quát anti-pattern, interaction và accessibility                     |
| Tech stack hỗ trợ              | 22 loại     | React, Next.js, Vue, Nuxt, SwiftUI, Flutter, JavaFX...                  |

Lấy “hãy làm landing page cho một spa làm đẹp” làm ví dụ, kết quả tạo ra trước tiên sẽ xếp trang vào bối cảnh health and wellness, sau đó đưa ra hướng Soft UI, dùng màu hồng nhạt, xanh sage và điểm nhấn vàng, chọn font Cormorant Garamond. Cùng một kết quả cũng liệt kê anti-pattern, chẳng hạn tránh gradient tím-hồng phổ biến.

Claude Code có thể cài từ marketplace của plugin:

```text
/plugin marketplace add nextlevelbuilder/ui-ux-pro-max-skill
/plugin install ui-ux-pro-max@ui-ux-pro-max-skill
```

Các công cụ như Codex, Cursor và Windsurf phù hợp hơn với CLI chính thức hiện tại. Tên package là `ui-ux-pro-max-cli`, `uipro-cli` cũ đã không còn được cập nhật:

```bash
npx ui-ux-pro-max-cli init --ai codex
npx ui-ux-pro-max-cli init --ai cursor
```

Script tra cứu của nó phụ thuộc vào Python 3. Sau khi cài đặt, chỉ cần mô tả loại trang và ngành là có thể kích hoạt:

```text
Hãy làm landing page cho một sản phẩm SaaS
Thiết kế một dashboard phân tích y tế
Làm một app tài chính với dark theme
```

Delivery check sẽ tiếp tục kiểm tra icon, trạng thái hover, độ tương phản văn bản và `reduced-motion`, trong đó có các quy tắc rõ ràng như “không dùng emoji làm icon”. Project đã có design system cần thận trọng khi sử dụng đề xuất tự động, nếu không color palette, font và quy chuẩn component được tạo mới có thể xung đột với quy tắc hiện có. Trong trường hợp này, viết quy chuẩn thiết kế sẵn có của team thành Skill cấp project sẽ phù hợp hơn.

Địa chỉ project: <https://github.com/nextlevelbuilder/ui-ux-pro-max-skill>

Khi không cần knowledge base thiết kế đầy đủ, có thể đổi sang Skill **frontend-design** chính thức của Anthropic.

Phiên bản hiện tại nhấn mạnh hơn vào việc xác định hướng visual rõ ràng trước, sau đó hiện thực hóa hướng đó qua font, màu sắc, layout, animation và chi tiết, đồng thời tránh cảm giác “AI page” rập khuôn. Nó không còn coi việc cấm Inter, Roboto hoặc gradient tím-trắng là quy tắc cố định; có dùng font và màu sắc nào hay không nên do brand guideline, loại nội dung và design system hiện có quyết định.

## Vercel React Best Practices

Hiện nay không khó để Agent viết ra một trang React có thể chạy; điều dễ bị bỏ qua là vấn đề performance phía sau: vài request vốn có thể chạy song song lại bị viết thành tuần tự, client nhận một lượng lớn dữ liệu không dùng đến, component re-render nhiều lần vì xử lý dependency không phù hợp, một package tiện tay được thêm vào lại làm Bundle phình to.

**vercel-react-best-practices** chính thức của Vercel tập trung vào những vấn đề này. Tính đến tháng 7 năm 2026, tổng số lượt cài đặt của nó trên [skills.sh](https://skills.sh/vercel-labs/agent-skills/vercel-react-best-practices) khoảng 570 nghìn, bao gồm 70 quy tắc performance cho React và Next.js, được chia thành 8 nhóm theo mức độ ảnh hưởng.

| Mức ưu tiên | Nội dung kiểm tra chính                                                                                                   |
| ----------- | ------------------------------------------------------------------------------------------------------------------------- |
| CRITICAL    | Loại bỏ request waterfall, kiểm soát kích thước Bundle                                                                    |
| HIGH        | Server-side cache và request deduplication, lấy dữ liệu song song, giảm chi phí serialization của React Server Components |
| MEDIUM      | Client-side request deduplication, quản lý dependency, giảm re-render không cần thiết, xử lý vấn đề Hydration             |
| LOW-MEDIUM  | DOM batching, cache phép tính lặp lại, dùng Set hoặc Map tối ưu việc tra cứu thường xuyên                                 |

Mỗi quy tắc đều có error code, code sau khi sửa và điều kiện áp dụng. Khi viết trang mới, thực hiện performance review hoặc refactor code React, để Agent kiểm tra từng mục theo các quy tắc này sẽ cụ thể hơn việc nhắc tạm một câu “hãy tối ưu performance”.

Lệnh cài đặt như sau:

```bash
npx skills add https://github.com/vercel-labs/agent-skills --skill vercel-react-best-practices
```

Bộ quy tắc này chỉ phù hợp với project React và Next.js; project Vue, Svelte hoặc backend không cần cài. Tối ưu performance cũng không thể chỉ xem checklist quy tắc; với vấn đề liên quan đến cache, serialization và rendering, cuối cùng vẫn phải kết hợp build artifact, React Profiler và request chain thực tế để verify.

Địa chỉ project: <https://github.com/vercel-labs/agent-skills/tree/main/skills/react-best-practices>

## sanyuan-skills

`sanyuan-skills` hiện có 6 Skill độc lập, khi cài đặt có thể chọn theo thư mục. Nếu chỉ muốn bổ sung một lượt code review trước khi commit, chỉ cần cài Code Review Expert, không đồng thời đưa vào quy trình làm rõ yêu cầu, TDD và những quy trình khác.

| Skill              | Bối cảnh áp dụng                                                                       |
| ------------------ | -------------------------------------------------------------------------------------- |
| Code Review Expert | Review code từ các khía cạnh SOLID, security, performance, error handling và edge case |
| Sigma              | Học concept kỹ thuật thông qua việc đặt câu hỏi kiểu Socrates                          |
| Skill Review       | Kiểm tra structure, description, process và mức sử dụng Token của Skill                |
| Skill Forge        | Tạo Skill mới                                                                          |
| Wiki Ingest        | Sắp xếp bài viết, tài liệu hoặc ghi chú thành Wiki có thể cross-reference              |
| Book Study         | Hướng dẫn đọc, kiểm tra mức độ nắm vững và ôn tập cách quãng                           |

Java developer dễ sử dụng trực tiếp nhất là Code Review Expert. Nó phù hợp để bổ sung một lượt review độc lập trước khi commit, nhưng không thể thay thế quy tắc kiểm tra riêng của project. Transaction boundary, quy ước error code, field log và yêu cầu compatibility vẫn phải được đưa vào `AGENTS.md`, hướng dẫn review hoặc test của repo.

Mỗi Skill đều có thể cài riêng:

```bash
npx skills add sanyuan0704/sanyuan-skills --skill code-review-expert
npx skills add sanyuan0704/sanyuan-skills --skill sigma
npx skills add sanyuan0704/sanyuan-skills --skill skill-review
npx skills add sanyuan0704/sanyuan-skills --skill skill-forge
```

Sau khi cài có thể gọi trực tiếp:

```text
/code-review-expert    # Review thay đổi git hiện tại
/sigma <chủ đề>        # Bắt đầu hướng dẫn học, ví dụ /sigma React Hooks
/skill-review          # Kiểm tra Skill hiện có
/skill-forge           # Tạo Skill mới
```

Địa chỉ project: <https://github.com/sanyuan0704/sanyuan-skills>

## Supabase Postgres Best Practices

**supabase-postgres-best-practices** do Supabase chính thức duy trì, nội dung chủ yếu xoay quanh PostgreSQL, project PostgreSQL thông thường cũng có thể dùng. Tính đến tháng 7 năm 2026, tổng số lượt cài đặt của nó trên [skills.sh](https://skills.sh/supabase/agent-skills/supabase-postgres-best-practices) khoảng 300 nghìn.

Nó chia các quy tắc PostgreSQL thành 8 nhóm: query performance, connection management, security và RLS, thiết kế table, concurrency và lock, data access pattern, monitoring diagnostics và advanced features. Mỗi quy tắc giải thích nguyên nhân vấn đề xuất hiện, sau đó đưa ra SQL sai, SQL đã sửa, phân tích `EXPLAIN` và chỉ số performance.

Ví dụ, khi yêu cầu Agent review một SQL “query order theo user, status và thời gian tạo”, nó sẽ tiếp tục kiểm tra thứ tự field của composite index, điều kiện query có hit index hay không, connection pool có hợp lý không, và thay đổi này có ảnh hưởng đến việc ghi cùng lock contention hay không, chứ không chỉ để lại một câu “thêm index”.

Lệnh cài đặt như sau:

```bash
npx skills add https://github.com/supabase/agent-skills --skill supabase-postgres-best-practices
```

Project Java backend chỉ cần sử dụng PostgreSQL là có thể dùng trực tiếp để kiểm tra SQL, table structure và database config. Project MySQL không nên áp dụng nguyên xi RLS, index và database parameter trong đó; khi tạo index, sửa field hoặc điều chỉnh connection pool, còn phải kết hợp data volume, `EXPLAIN ANALYZE` và kết quả load test để đánh giá, không thể để Agent trực tiếp sửa production database.

Địa chỉ project: <https://github.com/supabase/agent-skills/tree/main/skills/supabase-postgres-best-practices>

## Claude API

Skill Claude API hướng đến phát triển AI application. Nếu chỉ sử dụng Coding Agent trong IDE thì có thể bỏ qua; khi làm intelligent customer service, platform code generation, công cụ phân tích tài liệu hoặc platform Agent nội bộ, nó có thể dùng để đối chiếu chi tiết SDK và API.

Model name, parameter và streaming event đều có thể thay đổi theo version; viết code call dựa trên trí nhớ rất dễ dùng phải interface cũ. Skill **claude-api** chính thức của Anthropic bao quát model selection, pricing, parameter, streaming output, tool calling, MCP, Agent, cache, Token calculation và model migration, đồng thời chia tài liệu theo ngôn ngữ:

| Ngôn ngữ / cách tích hợp | Mô tả                                          |
| ------------------------ | ---------------------------------------------- |
| **Python**               | Dùng official Python SDK                       |
| **TypeScript**           | Dùng official TypeScript SDK và Zod            |
| **Java**                 | Project Java / Kotlin / Scala có thể tham khảo |
| **Go**                   | Có thể tham khảo cho ứng dụng server-side Go   |
| **Ruby / PHP**           | Phù hợp với project dùng stack tương ứng       |
| **C#**                   | Project .NET có thể tham khảo                  |
| **cURL**                 | HTTP raw, Shell script hoặc dùng để debug      |

Khi gặp tên method của SDK, parameter, streaming event hoặc structure của tool call, Skill này sẽ tra tài liệu tương ứng trước rồi mới tạo code. Tác dụng chính của nó là giảm tình trạng Agent đoán interface dựa trên memory của version cũ.

Địa chỉ project: <https://github.com/anthropics/skills/tree/main/skills/claude-api>

## skill-creator

Đây là meta-skill trong repo Skills chính thức của Anthropic, dùng để tạo, sửa và đánh giá Skill.

Nó cung cấp một quy trình phát triển Skill:

| Giai đoạn                | Nội dung công việc                                                                   |
| ------------------------ | ------------------------------------------------------------------------------------ |
| **Nắm bắt ý định**       | Hiểu bạn muốn Skill làm gì, xác định rõ ranh giới và mục tiêu                        |
| **Soạn SKILL.md**        | Viết file instruction cốt lõi của Skill, bao gồm frontmatter và nội dung instruction |
| **Test và verification** | Tạo test case, chạy thử nghiệm so sánh (có Skill và không có Skill)                  |
| **Iteration và tối ưu**  | Liên tục cải thiện instruction theo feedback của test                                |
| **Tối ưu description**   | Tối ưu description của Skill để tăng độ chính xác khi trigger                        |

Nó còn có công cụ evaluation, có thể so sánh output “dùng Skill” và “không dùng Skill”, ghi lại thời gian, Token và kết quả assertion, sau đó tạo báo cáo trực quan. Nếu không có Skill mà model đã có thể ổn định hoàn thành task, thì không cần tiếp tục duy trì một quy trình bổ sung.

Phù hợp làm điểm bắt đầu cho developer muốn xây dựng Skill riêng cho team.

Địa chỉ project: <https://github.com/anthropics/skills/tree/main/skills/skill-creator>

## Chọn thế nào

| Vấn đề bạn thường xuyên gặp phải                                     | Ưu tiên cân nhắc                 | Trường hợp không nên cài                                                         |
| -------------------------------------------------------------------- | -------------------------------- | -------------------------------------------------------------------------------- |
| Feature phức tạp dễ sót yêu cầu, sót test                            | Superpowers                      | Chủ yếu xử lý thay đổi nhỏ, Agent hiện tại đã có thể hoàn thành ổn định          |
| Chỉ muốn bổ sung làm rõ yêu cầu, chẩn đoán Bug, TDD hoặc code review | mattpocock/skills                | Project đã có quy trình tương đương ổn định                                      |
| Team muốn thống nhất Agent, Hook, memory và security strategy        | ECC                              | Chỉ thiếu một quy trình code review hoặc TDD                                     |
| PRD, technical proposal thường viết không rõ                         | Doc Co-Authoring                 | Chỉ sửa một đoạn nhỏ của tài liệu hiện có                                        |
| Không có design system, page do AI tạo thường na ná nhau             | UI UX Pro Max                    | Project đã có component library và quy chuẩn thiết kế hoàn thiện                 |
| Page React / Next.js chạy được nhưng có nhiều vấn đề performance     | Vercel React Best Practices      | Project không dùng React                                                         |
| Muốn bổ sung một lượt code review chung trước khi commit             | Code Review Expert               | Rủi ro của project chủ yếu đến từ quy tắc nội bộ, kiểm tra chung không giúp được |
| Query và table structure PostgreSQL thường phải làm lại              | Supabase Postgres Best Practices | Project dùng MySQL hoặc database khác                                            |
| Đang phát triển Claude API application                               | Claude API                       | Chỉ sử dụng Coding Agent trong IDE                                               |
| Muốn đúc kết task thường xuyên thất bại thành Skill                  | skill-creator                    | Chưa thực hiện baseline test khi không có Skill                                  |

Cách làm của mình là trước tiên chạy một lần task cùng loại mà không dùng Skill. Nếu model vốn đã có thể hoàn thành ổn định thì không cài; khi cùng một vấn đề liên tục xuất hiện, mới xem `SKILL.md`, `scripts/` và `references/` của project ứng viên để xác nhận nó thật sự lấp được khoảng trống, đồng thời không có command nguy hiểm hoặc permission quá rộng.

Lần đầu cài đặt nên giới hạn trong repo hiện tại. Sau khi chạy liên tục hai, ba task thực tế, hãy so sánh số lần làm lại, thời gian thực thi và độ ổn định của kết quả. Nếu không có cải thiện rõ ràng thì xóa; chỉ cân nhắc bật global sau khi xác nhận có ích lâu dài.

Cụ thể nên bắt đầu từ đâu phụ thuộc vào loại vấn đề gần đây thường phải làm lại nhất: nếu code review luôn bỏ sót cùng một loại rủi ro, có thể thử Code Review Expert; nếu Agent bắt đầu làm khi yêu cầu còn chưa chốt, có thể thử Doc Co-Authoring, hoặc [`grilling`](https://github.com/mattpocock/skills/blob/main/skills/productivity/grilling/SKILL.md) nhẹ hơn.

Danh sách này không có mục nào mặc định bắt buộc phải cài. Chỉ khi có thể nói rõ một Skill giải quyết vấn đề gì, trigger khi nào và xóa thế nào sau khi không còn hiệu quả, nó mới đáng được giữ lại trong danh sách.
