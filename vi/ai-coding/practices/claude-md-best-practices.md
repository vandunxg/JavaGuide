---
title: "CLAUDE.md: Thực hành tốt nhất: nên viết gì, không nên viết gì và tách thế nào khi dự án lớn lên"
description: "Giải thích những quy tắc dự án nào phù hợp để ghi trong CLAUDE.md, cũng như cách phối hợp với .claude/rules và Auto Memory để quản lý, duy trì các quy tắc này."
category: AI Coding thực chiến
head:
  - - meta
    - name: keywords
      content: CLAUDE.md,Claude Code,AI Coding,quy ước dự án AI,Agentic Coding,phát triển có AI hỗ trợ,thực hành tốt nhất CLAUDE.md,.claude/rules
---

Xin chào, tôi là Tiểu G. Vài ngày trước, khi chia sẻ [mẹo sử dụng Claude Code](https://javaguide.cn/ai-coding/practices/claudecode-tips.html), tôi đã giới thiệu sơ lược về `CLAUDE.md`. Có bạn đọc G hỏi trong phần bình luận rằng liệu có thể viết riêng một bài về file này hay không.

Nhiều người lần đầu thấy `CLAUDE.md` sẽ xem nó như một README khác. README chủ yếu giới thiệu dự án cho con người, còn `CLAUDE.md` cung cấp hướng dẫn làm việc cho Claude Code, chẳng hạn dự án khởi động thế nào, file nào không được sửa, format response của API là gì, sửa code xong cần chạy những kiểm tra nào.

Những nội dung này tất nhiên có thể được giải thích lại trong mỗi session mới, nhưng rất dễ bỏ sót. Các quy tắc có hiệu lực lâu dài mà Claude không thể suy ra chính xác từ source code phù hợp hơn để ghi sẵn vào `CLAUDE.md`.

Bài viết này kết hợp [tài liệu chính thức của Claude Code](https://code.claude.com/docs/en/best-practices) với kinh nghiệm sử dụng của tôi để giới thiệu nên viết gì trong `CLAUDE.md`, phân tách thế nào và duy trì ra sao về sau.

Một lưu ý nhỏ: Claude Code được cập nhật rất nhanh, đặc biệt `.claude/rules/` và Auto Memory rất dễ thay đổi theo version. Bài viết này được đối chiếu với tài liệu chính thức ngày 2026-07-24. Trước khi áp dụng thực tế, hãy dùng `claude --version` để xác nhận version, sau đó dùng `/context` để xem các instruction thực tế được load trong session hiện tại; `/memory` chủ yếu dùng để xem và chỉnh sửa vị trí cấu hình của rule, memory.

## CLAUDE.md là gì?

`CLAUDE.md` là file instruction lâu dài của Claude Code, có thể được đặt ở cấp user, cấp project và các vị trí khác. Sau khi đọc file này, Claude Code sẽ xử lý project hiện tại dựa trên các command, quy ước và giới hạn trong đó.

Các nội dung phù hợp để ghi vào gồm:

- Các quy tắc Claude dễ đoán sai
- Các quy ước không thể đọc ra từ code
- Các quy chuẩn team bắt buộc tuân thủ
- Version của tech stack, command thường dùng, quyết định kiến trúc, vấn đề đặc thù của project

Để phán đoán một nội dung có cần giữ lại hay không, hãy hỏi:

> Nếu xóa dòng này, Claude có dễ mắc lỗi hơn không?

Nếu có thì giữ lại; nếu không, nhiều khả năng nó chỉ đang lãng phí context.

## CLAUDE.md khác gì với các file rule khác?

![Phân công giữa CLAUDE.md và các file rule khác](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/claude-md-best-practices-rule-files-relationship.png)

### CLAUDE.md vs AGENTS.md

|             | CLAUDE.md                                  | AGENTS.md                                                                                 |
| ----------- | ------------------------------------------ | ----------------------------------------------------------------------------------------- |
| **Ai đọc**  | Chỉ Claude Code                            | Tiêu chuẩn mở liên tool, OpenAI Codex, Cursor, Google Jules và các tool khác cũng sử dụng |
| **Định vị** | File rule của project dành cho Claude Code | File instruction Agent dùng chung giữa các tool                                           |

![CLAUDE.md và AGENTS.md](https://oss.javaguide.cn/github/javaguide/ai/coding/claude-agents-md.png)

**AGENTS.md** hướng tới nhiều coding Agent, còn **CLAUDE.md** là entry point dành riêng cho Claude Code. Khi repository sử dụng đồng thời hai loại file, có thể để chúng dùng chung một bộ instruction cơ sở.

Team cũng có thể quy ước một khu vực “các lỗi thường gặp đã được xác nhận” trong `AGENTS.md`, nhưng Agent sẽ không tự động ghi lỗi chỉ vì file tồn tại, càng không thể đảm bảo rằng sau khi thêm một rule thì lần sau sẽ không mắc lại. Việc có ghi vào hay không, ai review và khi nào xóa phải được làm rõ trong workflow.

Nếu repository đã dùng `AGENTS.md` để cung cấp instruction cho các coding Agent khác, có thể tạo một `CLAUDE.md` import `AGENTS.md`, để hai tool dùng chung instruction cơ sở mà không phải duy trì lặp lại.

```markdown
@AGENTS.md

## Instruction riêng cho Claude Code

- Dùng plan mode để xử lý các thay đổi trong `src/billing/`
```

Bài viết [Giải thích Harness Engineering trong một bài](https://javaguide.cn/ai/agent/harness-engineering.html) của tôi cũng từng giới thiệu một ví dụ: `AGENTS.md` của OpenAI chỉ khoảng 100 dòng, chủ yếu trỏ tới các tài liệu thiết kế cụ thể hơn, sơ đồ kiến trúc, kế hoạch thực thi và đánh giá chất lượng trong thư mục docs/. Agent trước tiên đọc file entry point, đến khi xử lý task liên quan mới load tài liệu chi tiết, tránh đưa toàn bộ nội dung vào context ngay từ đầu.

### CLAUDE.md vs .claude/rules/

|                          | CLAUDE.md                                                                                                                                  | `.claude/rules/`                                                                                  |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------- |
| **Cách load**            | File ở root/cấp project thường được load khi session khởi động; file trong subdirectory được load theo nhu cầu khi đọc directory tương ứng | Rule không có `paths` được load khi khởi động; rule có `paths` được load khi Claude đọc file khớp |
| **Trường hợp dùng**      | Rule dùng chung trên toàn project                                                                                                          | Rule chỉ áp dụng cho file/directory cụ thể                                                        |
| **Mức tiêu thụ context** | Rule ở root/cấp project liên tục tiêu thụ context                                                                                          | Chỉ path-scoped rules tiêu thụ theo nhu cầu; global rules vẫn liên tục tiêu thụ context           |

Các rule như quy chuẩn backend API, cấu hình test chỉ có hiệu lực trong directory cụ thể có thể đặt vào `.claude/rules/`, không cần tiếp tục ghi vào `CLAUDE.md` ở root.

Cần chú ý hai điểm:

1. `.claude/rules/` không phải directory chắc chắn xuất hiện mặc định sau khi cài Claude Code, có thể tạo thủ công khi cần.
2. Path rule có frontmatter `paths` được load theo kết quả match; rule không có `paths` vẫn được đưa vào context dưới dạng global rule. Vì vậy, chỉ tạo directory `.claude/rules/` không thể tự giảm mức tiêu thụ context.

### CLAUDE.md vs SPEC.md

|              | CLAUDE.md                                                        | SPEC.md                                                                                 |
| ------------ | ---------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| **Mục đích** | Quy tắc project (làm việc thế nào)                               | Đặc tả yêu cầu (làm gì)                                                                 |
| **Nội dung** | Quy chuẩn coding, command thường dùng, ghi chú lỗi, quy ước team | Phạm vi yêu cầu, định nghĩa tính năng, tiêu chuẩn nghiệm thu, tương tự PRD hướng tới AI |
| **Ai dùng**  | Trợ lý coding AI (coding hằng ngày)                              | Quy trình Spec Coding (phát triển hướng theo yêu cầu)                                   |

`SPEC.md` là tên file được một số team sử dụng trong **Spec Coding**, còn `Specify → Design → Implement → Test` là một cách tổ chức được dùng trong bài viết này. Các tool khác có thể dùng các file và phase như requirements, design, tasks, plan; không có một tiêu chuẩn bốn phase thống nhất bắt buộc.

![Pipeline coding hướng theo đặc tả Spec Coding](https://oss.javaguide.cn/github/javaguide/ai/coding/spec-coding-pipeline-flow.png)

`requirements.md` trong hình trên là file yêu cầu được tạo ở phase `Specify` của workflow này; các team khác cũng có thể tập trung đặc tả task tương tự vào `SPEC.md`, hai tên này không phải alias cố định dùng chung.

Tôi đã giới thiệu chi tiết trong bài [Thực chiến Spec Coding hướng theo đặc tả: từ Vibe Coding đến quy chuẩn code AI](https://javaguide.cn/ai-coding/practices/spec-coding.html).

Có thể phân biệt như sau: **CLAUDE.md quản lý quy chuẩn hành vi dài hạn, còn Spec quản lý ràng buộc của task hiện tại.**

### Nên chọn thế nào trong thực tế?

- **CLAUDE.md**: quy chuẩn hành vi dành riêng cho Claude Code; cấp root/user thường được load khi session bắt đầu, rule ở subdirectory có hiệu lực theo nhu cầu.
- **AGENTS.md**: rule “làm việc thế nào” dùng chung giữa các tool, có thể được `CLAUDE.md` import để tái sử dụng.
- **`.claude/rules/`**: directory rule cục bộ; không có `paths` thì gần giống global rule, có `paths` thì chỉ có hiệu lực khi xử lý file khớp.
- **SPEC.md**: file đặc tả yêu cầu, định nghĩa việc cần làm trong lần này, là một phần của workflow Spec Coding.

## Rốt cuộc nên viết gì trong CLAUDE.md?

Hãy xem một cách viết mà tôi thường gặp. Nhiều người chạy xong `/init`, thấy Claude tạo một `CLAUDE.md`, nghĩ rằng “có còn hơn không”, rồi gần như commit nguyên trạng:

```markdown
# Giới thiệu project

Đây là project Spring Boot, sử dụng Java 17 và Maven.

# Code style

- Viết code sạch
- Tuân thủ best practices
- Đảm bảo code dễ đọc

# Workflow

- Chạy test trước khi commit
- Duy trì tổ chức code tốt
```

Bản thân các yêu cầu này không sai, nhưng rất khó làm thay đổi hành vi của Claude. Lấy “viết code sạch” làm ví dụ, sau khi xóa câu này Claude vẫn sẽ cố gắng tạo code dễ đọc. Giữ nó trong file chỉ tiếp tục chiếm context.

`CLAUDE.md` cùng với system instruction, lịch sử hội thoại và các file đã đọc sẽ cùng chiếm context. Anthropic chỉ ra trong tài liệu chính thức rằng: **khi context window được lấp đầy, hiệu suất tổng thể của Claude sẽ giảm.**

![Vì sao context mất tác dụng](https://oss.javaguide.cn/github/javaguide/ai/context-engineering/why-does-the-following-content-fail.png)

File càng dài, không gian dành cho hội thoại và code về sau càng ít, các rule thực sự quan trọng cũng càng dễ bị những nội dung khác lấn át.

Anthropic khuyến nghị giữ `CLAUDE.md` ngắn gọn, không quá 200 dòng, chỉ giữ lại những thông tin Claude không thể dễ dàng suy ra từ code. Nếu nội dung tiếp tục phình to, có thể tách sang `.claude/rules/` có `paths`, hoặc đưa các nội dung tham khảo không cần trong mọi session vào Skills.

![Khuyến nghị của tài liệu chính thức Claude Code về CLAUDE.md](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/claudemd-claude-docs.png)

Khi kiểm tra `CLAUDE.md`, có thể hỏi từng dòng: “Sau khi xóa dòng này, Claude có dễ mắc cùng loại lỗi hơn không?” Giữ lại các rule có ảnh hưởng rõ ràng; xóa trước những nội dung không cho thấy khác biệt về hành vi, sau này gặp vấn đề thực tế thì bổ sung.

### Những nội dung nên viết

Có năm nhóm nội dung chính đáng để viết vào `CLAUDE.md`.

**1\. Thông tin tech stack và version.**

Khác biệt version của framework thường ảnh hưởng trực tiếp đến kết quả sinh code. Ví dụ, một số cách cấu hình của Spring Boot 2 và 3 khác nhau; nếu không nói rõ version, Claude có thể sinh cách dùng không khớp với project hiện tại. Những dependency như MyBatis-Plus có thể đọc từ `pom.xml`, nhưng lý do chọn nó thay vì dùng JPA thì cần giải thích thêm.

**2\. Command thường dùng.**

Ghi trực tiếp command build, test, lint và khởi động project vào `CLAUDE.md`. Dùng code block hoặc inline code để giữ tham số gốc, khi thực thi Claude không cần chuyển ngôn ngữ tự nhiên thành command lần nữa.

```markdown
# Commands

- Build: `mvn clean package -DskipTests`
- Test: `mvn test -pl module-name`
- Khởi động: `mvn spring-boot:run -pl bootstrap`
- Kiểm tra code: `mvn checkstyle:check`
```

**3\. Quyết định kiến trúc và lý do phía sau.**

Lý do phía sau một rule sẽ ảnh hưởng đến cách Claude xử lý các tình huống lân cận. Trước đây project của tôi chỉ viết “không được viết SQL trực tiếp, dùng QueryWrapper”, nhưng Claude vẫn viết SQL trong một số query. Sau đó tôi bổ sung lý do: “Hệ thống audit SQL phụ thuộc vào việc phân tích Wrapper để ghi log thao tác.” Phạm vi áp dụng của ràng buộc này trở nên rõ ràng, các query khác cũng nên dùng Wrapper.

**4\. Quy ước team và các vấn đề đặc thù của project.**

Format commit message (chẳng hạn `feat(scope): message`), quy tắc đặt tên branch, dependency của environment variable là những thứ khó xác định chỉ bằng cách đọc code. Những thông tin thành viên mới cần hỏi khi tiếp quản project cũng phù hợp để ghi vào `CLAUDE.md`.

**5\. Thông tin task cần giữ qua nhiều session.**

Khi cần giữ mô tả task, tiêu chuẩn nghiệm thu, mức ưu tiên, dependency và vấn đề blocking qua nhiều session, có thể tạo riêng một file task rồi để `CLAUDE.md` hoặc `AGENTS.md` trỏ tới file đó. Cách này vừa giữ được tiến độ task hiện tại, vừa không trộn nội dung thường xuyên thay đổi với rule dài hạn.

### Những nội dung không nên viết

**1\. Quy tắc code style.**

Indent dùng bao nhiêu space, import sắp xếp thế nào, có cần semicolon ở cuối hay không, hãy giao những việc này cho formatting tool.

Project chưa cấu hình Checkstyle hoặc Prettier thì hãy cấu hình tool trước, đừng dùng ngôn ngữ tự nhiên để làm việc format code.

**2\. Hành vi mặc định của ngôn ngữ hoặc framework.**

Ví dụ:

- “Vue dùng `ref` / `reactive` để quản lý reactive state.”
- “Entity JPA tương ứng với table database.”
- “SQL dùng `WHERE` để lọc điều kiện.”

Đây hoàn toàn là những điều hiển nhiên, viết ra chỉ làm tăng gánh nặng hiểu cho AI.

**3\. Tài liệu tham khảo dài.**

Đừng nhét nguyên đoạn tài liệu API bên ngoài, bảng tham số SDK vào đây. Chỉ cần đặt link, khi thực sự cần dùng Claude sẽ đọc.

### Ví dụ CLAUDE.md tốt

#### Ví dụ cấp user: trước tiên kiểm soát thói quen xấu phổ biến

[andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills) là một project rule/Skills Claude Code do bên thứ ba tổng hợp, lấy cảm hứng từ các quan sát công khai của Andrej Karpathy về vấn đề coding với LLM. Các rule trong đó không phụ thuộc repository cụ thể, phù hợp hơn để đặt ở cấp user nhằm hạn chế các hành vi lệch hướng thường gặp trong quá trình coding.

Hình dưới đây là ví dụ `CLAUDE.md` trong repository này:

![andrej-karpathy-claudemd](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/andrej-karpathy-claudemd.png)

File này chỉ xử lý một vài vấn đề thường gặp: kiểm tra giả định trước khi coding, tránh abstraction quá mức, giới hạn thay đổi trong phạm vi task, chạy test sau khi hoàn tất và đối chiếu với tiêu chuẩn nghiệm thu. Số lượng rule không nhiều, mỗi rule đều tương ứng với một hành vi có thể quan sát.

Bạn nên đọc trực tiếp bản gốc trên GitHub, ở đây không trích dẫn nguyên đoạn nữa.

#### Ví dụ cấp project: viết quy tắc repository thành thẻ tra nhanh

[interview-guide](https://javaguide.cn/zhuanlan/interview-guide.html) của tôi sử dụng `CLAUDE.md` cấp project. File ở root giữ tech stack, command thường dùng, ranh giới phân layer, xử lý exception, rule transaction và danh sách cấm; quy chuẩn chi tiết giao cho `.claude/rules/`.

File ở root sau khi tinh gọn có thể viết như sau:

```markdown
# AI Interview Platform Rules

Nền tảng phỏng vấn Spring Boot 4.0 + Java 21 + Spring AI 2.0 + React.

## Tech Stack

- Backend: Spring Boot 4.0 / Java 21 / Gradle / Spring AI 2.0
- Database: PostgreSQL + pgvector (1024 chiều COSINE)
- Cache & MQ: Redis / Redisson / Redis Stream
- Frontend: React 18 + TypeScript + Vite + TailwindCSS 4 (`frontend/`)
- Mapping & Docs: MapStruct / OpenAPI / iText 8 / Apache Tika

## Commands

- Build: `./gradlew build`
- Test: `./gradlew test`
- Khởi động backend: `./gradlew bootRun`
- Khởi động frontend: `cd frontend && npm run dev`
- Kiểm tra frontend: `cd frontend && npm run lint`

## Architecture

- Project Gradle đơn module, chia package theo tính năng.
- Backend tuân theo phân layer `Controller -> Service -> Repository`.
- Năng lực infrastructure đặt trong `common/`, gồm rate limiting, AI call, async task, cấu hình, exception, response thống nhất.
- Code frontend đặt trong `frontend/`.
- Xem cấu trúc project chi tiết tại `docs/architecture.md`.

## Must Follow

- Controller chỉ thực hiện validate parameter và bọc response, không viết business logic.
- Service đảm nhiệm điều phối business, chỉ đặt `@Transactional` ở layer Service.
- Repository chỉ phụ trách data access, không viết business logic.
- Response bên ngoài thống nhất dùng `Result<T>`.
- Business exception bắt buộc dùng `BusinessException(ErrorCode.XXX, "mô tả")`.
- Mapping Entity sử dụng MapStruct, cấm tự viết logic chuyển đổi lặp lại.
- Không đặt LLM, S3, external HTTP call bên trong database transaction.
- Thống nhất lấy `ChatClient` thông qua `LlmProviderRegistry`.
- Structured output thống nhất dùng `StructuredOutputInvoker` để bọc retry.
- Redis Stream producer/consumer sử dụng template `AbstractStreamProducer` / `AbstractStreamConsumer`.
- Rate limiting sử dụng `@RateLimit`, không tự viết logic Redis rate limiting rải rác.
- Vector search database sử dụng PostgreSQL + pgvector, dimension là 1024, distance type là COSINE.

## Never Do

- Không `throw new RuntimeException(...)`, bắt buộc dùng `BusinessException`.
- Không trả trực tiếp Entity cho frontend.
- Không rải `@Value` trong Service, tập trung cấu hình vào `@ConfigurationProperties`.
- Không inline fully qualified class name, hãy dùng import.
- Không gọi LLM, S3 hoặc external HTTP trong transaction.
- Không gọi method `@Transactional` nội bộ cùng class.
- Không `catch (Exception e) {}` rồi bỏ qua một cách im lặng.
- Không gọi DB trong loop, ưu tiên batch operation.
- Không hard-code secret.
- Không sử dụng `Executors.newXxxThreadPool()`, sử dụng `ThreadPoolExecutor` tường minh.

## More Rules

- Quy chuẩn error code: `.claude/rules/error-handling.md`
- Quy chuẩn rate limiting: `.claude/rules/rate-limit.md`
- Quy chuẩn Redis Stream: `.claude/rules/redis-stream.md`
- Quy chuẩn AI service call: `.claude/rules/ai-service.md`
- Quy chuẩn database: `.claude/rules/database.md`
- Quy chuẩn frontend: `.claude/rules/frontend.md`
```

## Viết thế nào để Claude thực sự tuân thủ?

Rule cần tương ứng với action rõ ràng và có thể kiểm tra kết quả thực thi.

### Rule phải cụ thể và có thể xác minh

“Chú ý khả năng đọc của code” không đưa ra tiêu chuẩn có thể kiểm tra. Đổi thành “Tên function bắt đầu bằng verb, mỗi function không quá 40 dòng”, khi đó Claude mới biết phải làm gì cụ thể, lúc code review cũng có thể đối chiếu trực tiếp.

### Lệnh cấm phải đi kèm phương án thay thế

Lệnh cấm tốt nhất nên đồng thời đưa ra phương án thay thế: không làm X, gặp tình huống này thì dùng Y.

Hãy lấy một ví dụ trong project của tôi. Trước đây Claude thường viết field injection bằng `@Autowired`, nhưng quy chuẩn team là constructor injection.

Ban đầu tôi chỉ viết “không dùng field injection bằng `@Autowired`”. Hiệu quả khá hạn chế: nó quả thật không dùng field injection nữa, nhưng đôi lúc chuyển sang tự viết constructor, đôi lúc lại dùng cách injection khác. Sau đó tôi hoàn thiện rule:

```markdown
# Dependency injection

- Không dùng field injection bằng @Autowired
- Dùng constructor injection, kết hợp với @RequiredArgsConstructor của Lombok
- Ví dụ tham khảo: cách viết trong UserController.java
```

Rule sau khi hoàn thiện đồng thời đưa ra cách viết được khuyến nghị và file tham khảo trong project, Claude có một mẫu có thể tuân theo trực tiếp khi xử lý code cùng loại.

### Dùng từ đánh dấu hợp lý, đừng lạm dụng

Khi Claude liên tục vi phạm một rule, có thể thêm `IMPORTANT:` hoặc `YOU MUST:` ở phía trước. Các từ đánh dấu này chỉ dùng cho một số ít rule quan trọng; nếu rule nào cũng có thì mức nhấn mạnh sẽ mất đi sự phân biệt.

Nếu Claude liên tục bỏ qua một rule, lúc này cần kiểm tra file có quá dài hay không, rule có được đặt đúng vị trí hay không. Tiếp tục thêm từ nhấn mạnh thường không giải quyết được hai vấn đề này.

### Dùng tên thông dụng cho heading

Các heading thông dụng như Commands, Structure, Conventions, Testing đã có thể mô tả chính xác nội dung. Claude dễ tìm command, cấu trúc directory và yêu cầu test hơn, thành viên project cũng không phải hiểu lại một bộ tên gọi khác.

### Hooks

`CLAUDE.md` chỉ có thể cung cấp instruction, không thể bắt buộc ngăn một lần gọi tool. Các thao tác như sửa file nhạy cảm, thực thi command nguy hiểm có thể giao cho Hook kiểm tra. Ví dụ, `PreToolUse` nhận thông tin invocation trước khi tool thực thi, sau đó quyết định cho phép hay từ chối dựa trên quyền hạn và rủi ro.

Quy trình thực thi do tài liệu chính thức Claude Code đưa ra như sau:

![Claude Code PreToolUse Hook](https://oss.javaguide.cn/github/javaguide/ai/coding/claude-code-runs-rm-rf-tmp-build-what-happens.svg)

Các yêu cầu có thể kiểm tra bằng máy nên ưu tiên giao cho Linter, Hook hoặc CI. Ví dụ, để ngăn Claude sửa `.env`, có thể để `PreToolUse` Hook kiểm tra target path và từ chối thao tác; chỉ nhắc trong `CLAUDE.md` vẫn có thể bỏ sót.

Các việc phù hợp để làm bằng Hook:

- Tự động format sau khi edit.
- Chạy test trước khi session kết thúc.
- Cấm sửa `migrations/` hoặc `.github/workflows/`.
- Chặn `curl | bash`, `rm -rf`, gửi nội dung nhạy cảm tới endpoint bên ngoài.
- Inject context bổ sung khi Sub-Agent khởi động.

Các yêu cầu mà chỉ cần bỏ sót một lần đã tạo ra rủi ro rõ ràng phù hợp để bắt buộc kiểm tra bằng Hook. Những nội dung chỉ nhằm giúp Claude hiểu quy ước của project thì tiếp tục viết trong `CLAUDE.md`.

## Đặt CLAUDE.md ở đâu?

Claude Code hỗ trợ đặt `CLAUDE.md` ở nhiều vị trí, phạm vi ảnh hưởng tương ứng như sau:

![Phân cấp và mức ưu tiên của CLAUDE.md](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/claude-md-best-practices-file-hierarchy.png)

| Vị trí           | Path                                                                                                                                                  | Mục đích                                                                                                                                       |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| **Cấp tổ chức**  | macOS: `/Library/Application Support/ClaudeCode/CLAUDE.md`, Linux/WSL: `/etc/claude-code/CLAUDE.md`, Windows: `C:\Program Files\ClaudeCode\CLAUDE.md` | Quy chuẩn coding, yêu cầu compliance và hướng dẫn xử lý dữ liệu được IT/DevOps phân phối thống nhất, không thể loại trừ bằng cấu hình cá nhân. |
| **Cấp user**     | `~/.claude/CLAUDE.md`                                                                                                                                 | Preference cá nhân, có hiệu lực với mọi project                                                                                                |
| **Cấp project**  | `./CLAUDE.md` hoặc `./.claude/CLAUDE.md`                                                                                                              | Quy chuẩn dùng chung của team, commit vào Git                                                                                                  |
| **Cấp local**    | `./CLAUDE.local.md`                                                                                                                                   | Cấu hình đặc thù project của cá nhân, thêm vào `.gitignore`                                                                                    |
| **Subdirectory** | `./subdir/CLAUDE.md`                                                                                                                                  | Load theo nhu cầu khi Claude truy cập file trong directory đó, không inject lúc session bắt đầu                                                |

`CLAUDE.md` ở các cấp khác nhau sẽ được load cùng nhau, file phía sau không trực tiếp ghi đè toàn bộ nội dung phía trước. Rule càng gần project hiện tại và phạm vi tác động càng cụ thể thì càng sát với task hiện tại.

Ví dụ, rule cấp user yêu cầu dùng space để indent, nhưng rule cấp project lại yêu cầu dùng Tab, Claude nhiều khả năng sẽ dùng rule của project trong project đó. Tuy nhiên, instruction xung đột sẽ làm tăng tính không chắc chắn, khi phát hiện nên dọn dẹp ngay.

Cách làm của tôi khá đơn giản: commit `CLAUDE.md` cấp project vào Git, ghi các rule team đều phải tuân thủ; những preference chỉ liên quan đến bản thân, chẳng hạn muốn commit message trong project hiện tại dùng tiếng Việt, thì đặt vào `CLAUDE.local.md`, thêm file này vào `.gitignore`, đừng trộn thói quen cá nhân vào file của team.

## Khi project lớn lên, quản lý CLAUDE.md thế nào?

Project vừa và nhỏ thường chỉ cần một `CLAUDE.md`. Khi số module tăng lên, tiếp tục nhồi mọi rule vào file ở root sẽ khiến mỗi session đều load nhiều nội dung không liên quan đến task hiện tại.

![Cách tổ chức CLAUDE.md theo quá trình phát triển](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/claude-md-best-practices-scaling-evolution.png)

### Khi project chưa lớn: chỉ giữ một file

Thông tin global như tech stack, command thường dùng và ranh giới kiến trúc giữ trong file ở root. Phần lớn project vừa và nhỏ chỉ cần một layer này, `CLAUDE.md` của riêng tôi thường cũng không quá 50 dòng.

### Khi nội dung nhiều lên: để file chính phụ trách routing

`CLAUDE.md` ở root giữ phần tổng quan project và command thường dùng, quy chuẩn kiến trúc, quy ước API, yêu cầu test đặt trong các file riêng, sau đó dùng `@path/to/file` để reference:

```markdown
## Project

Service quản lý order dùng Spring Boot 3.2 + MyBatis-Plus + MySQL 8.0.

## Commands

- Build: `mvn clean package`
- Test: `mvn test`

## Rules

- Quy ước API: @docs/api-conventions.md
- Quy chuẩn database: @docs/database-rules.md
```

### Khi rule chỉ có hiệu lực với một phần code: load theo path

Sử dụng frontmatter match file path trong `.claude/rules/`:

```markdown
---
paths:
  - "src/main/java/**/controller/**/*.java"
---

# Quy chuẩn Controller

- Thống nhất dùng Result<T> để bọc return value
- Mọi API phải thêm annotation Swagger
```

Khi edit Controller, rule Controller sẽ được load; khi xử lý Service thì phần context này không tiếp tục bị chiếm dụng. Trong thực tế, thời điểm load còn tạo ra ba ranh giới.

**Khi tạo file mới, path rule có thể chưa được load.** path-scoped rules được inject khi Claude **đọc** file khớp, không kiểm tra trước mỗi lần gọi tool. Khi trực tiếp tạo một file mới khớp path, rule ở giai đoạn tạo có thể chưa vào context. Những yêu cầu như “Controller mới phải có một file header nhất định” nên đặt trong global rules không có `paths`, `CLAUDE.md` ở root, hoặc giao cho Hook kiểm tra.

**Sau khi compact context, rule cục bộ cần được trigger lại.** Sau `/compact`, `CLAUDE.md` ở root sẽ được inject lại; `CLAUDE.md` ở subdirectory và path rule phải đợi Claude đọc lại file khớp thì mới được load. Trước khi tiếp tục sửa file, có thể dùng `/context` để kiểm tra instruction trong session hiện tại.

**Cần kiểm tra thực tế rule có được load hay không.** `/context` có thể xem `CLAUDE.md`, `CLAUDE.local.md` và rules file đã load trong session hiện tại; `/memory` dùng để xem và chỉnh sửa vị trí cấu hình. Khi cần ghi lại chi tiết quá trình load, có thể cấu hình `InstructionsLoaded` Hook.

Hiện tôi đang dùng chính tầng gồm file chính + rules file match theo path. Những cách nâng cao hơn (chẳng hạn import Skills và MCP để load capability động) vẫn đang được khám phá.

> **Gợi ý kỹ thuật**: `@path/to/file` sẽ embed toàn bộ file vào context. Reference một file vài trăm dòng sẽ khiến mỗi session ngay từ đầu đã chiếm nhiều không gian instruction. Tài liệu chính thức hiện giới hạn tối đa 4 tầng import đệ quy. Với file lớn, có thể viết lại thành “Chi tiết kiến trúc xem `docs/architecture.md`”, để Claude đọc khi cần.

## Duy trì thế nào?

Sau khi cấu trúc project, command và workflow thay đổi, các rule cũ trong `CLAUDE.md` cũng phải được dọn theo.

`CLAUDE.md` dùng để lưu các instruction dài hạn được chủ động duy trì, chẳng hạn rule team bắt buộc tuân thủ và sự thật về project mà mọi session đều phải biết. Auto Memory là cơ chế memory tự động tích hợp từ Claude Code v2.1.59+, có thể ghi lại kết luận debug, preference và thói quen làm việc xuất hiện trong quá trình phối hợp.

Thói quen của tôi là: nội dung ảnh hưởng đến việc phối hợp của team, session nào cũng phải tuân thủ thì ghi vào `CLAUDE.md`; kinh nghiệm nhỏ chỉ học được trong quá trình điều tra thì giao cho Auto Memory.

Ví dụ, “mọi API đều trả về `Result<T>`” nên ghi vào `CLAUDE.md`; còn “test Redis Stream của project này cần khởi động Redis local trước” là phát hiện debug, để Auto Memory ghi nhớ là đủ. Auto Memory mặc định được bật, có thể xem, chỉnh sửa, tắt trong `/memory`; nó duy trì directory memory riêng cho từng project, nhưng không phải quy chuẩn dùng chung của team và không thể thay thế `CLAUDE.md` được commit vào repository.

![Flow quyết định duy trì CLAUDE.md](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/claude-md-best-practices-maintenance-flow.png)

### Khi nào thêm rule?

Khi Claude đã từng mắc một loại lỗi nào đó và một rule rõ ràng có thể giảm xác suất lặp lại lỗi, lúc đó mới cân nhắc thêm. File rule chỉ có thể cung cấp instruction; các yêu cầu có thể kiểm tra bằng máy vẫn phải giao cho test, Linter, Hook hoặc CI.

### Khi nào xóa rule?

Khi ràng buộc tương ứng trong project không còn hiệu lực, hoặc sau khi xóa Claude không thay đổi hành vi, có thể xóa. Những sự thật đã có thể đọc trực tiếp trong code cũng không cần tiếp tục duy trì lặp lại trong file rule.

### Làm sao biết rule cần điều chỉnh?

Nếu Claude nói “Xin lỗi, vừa rồi tôi đã bỏ qua rule XX”, trước tiên hãy xác nhận file đó đã được load chưa, sau đó kiểm tra rule đã đưa ra action rõ ràng chưa. Khi rule chỉ yêu cầu “chú ý”, “cố gắng”, có thể đổi thành cách diễn đạt có thể thực thi, có thể xác minh.

Nếu cùng một rule liên tục bị vi phạm trong các session khác nhau, còn phải kiểm tra file có quá dài không, nhiều rule có xung đột không, rule có được đặt đúng scope không. Tiếp tục in đậm hoặc thêm dấu chấm than thường không thay đổi được các vấn đề này.

### Làm kiểm tra định kỳ thế nào?

Có thể định kỳ chọn vài rule và hỏi Claude: “Sau khi xóa rule này, bạn sẽ thay đổi những hành vi nào?” Câu trả lời này chỉ dùng để sàng lọc ban đầu, vì dự đoán của Claude về hành vi của chính nó không hoàn toàn đáng tin. Khi không chắc, có thể dùng hai session song song, lần lượt sử dụng `CLAUDE.md` có và không có rule đó, đưa cùng một prompt rồi so sánh kết quả.

Khi gặp lỗi, cũng đừng lập tức yêu cầu Claude ghi bài học vào `CLAUDE.md`. Trước tiên hãy phán đoán nó có hiệu lực lâu dài không, có ảnh hưởng đến thành viên team không, sau này có gặp lại không. Sau khi lỗi cùng loại lặp lại nhiều lần, hãy tổng hợp thành một rule; preference chỉ liên quan đến máy local hoặc kết luận debug một lần có thể giữ trong Auto Memory.

## Các lỗi thường gặp

- **Chỉ biết thêm rule mà quên xóa.** Mỗi lần xảy ra lỗi lại bổ sung một câu vào `CLAUDE.md`, còn rule cũ thì vẫn giữ, file sẽ nhanh chóng ngày càng dài. Đợi đến khi Claude bắt đầu bỏ sót rule rồi mới in đậm, thêm dấu chấm than thì không có nhiều ý nghĩa; hãy xóa nội dung hết hiệu lực, trùng lặp trước.
- **Dùng `@` để import file lớn trong một lần.** Reference bằng `@` sẽ đưa toàn bộ file vào context. Import trực tiếp tài liệu vài trăm dòng sẽ chiếm một phần không gian ngay cả khi session chưa bắt đầu. Với tài liệu ít dùng, viết “Xem chi tiết kiến trúc tại `docs/architecture.md`” là đủ, khi cần hãy để Claude tự đọc.
- **Đặt toàn bộ yêu cầu khi tạo file mới vào path-scoped rules.** Path rule phải đợi Claude đọc file khớp mới được load. Khi trực tiếp tạo file mới, các rule này có thể chưa vào context. Các yêu cầu bắt buộc tuân thủ trong giai đoạn tạo nên đặt trong global rules, `CLAUDE.md` ở root hoặc Hook.
- **Mỗi rule file nói một kiểu.** Khi rule cấp user, cấp project và cấp directory xung đột, Claude có thể không nhắc bạn, cũng không phải lần nào cũng lựa chọn giống nhau. Sau khi thay đổi rule của project, hãy tiện thể kiểm tra các tầng khác, dọn cả nội dung trùng lặp và xung đột.
- **Claude thỉnh thoảng mắc lỗi là lập tức thêm một rule vĩnh viễn.** Một vấn đề hiếm gặp đổi lấy một rule dài hạn khiến mỗi session sau đó đều phải tiêu tốn context cho nó. Khi vấn đề cùng loại lặp lại nhiều lần và thật sự có thể dùng một instruction rõ ràng để ràng buộc, thêm vào sau cũng chưa muộn.

## Tổng kết

Khi viết `CLAUDE.md`, không cần nghĩ đến việc nhét mọi quy tắc trong project vào đó, hoàn toàn không có ý nghĩa.

Một cách phán đoán tôi thường dùng là: Claude có thể đoán chính xác thông tin này chỉ bằng cách nhìn code không? Nếu không thể nhìn ra, sau khi đoán sai lại dễ làm lệch task, thì đó là loại thông tin nên viết.

Version tech stack, command khởi động và test, ranh giới kiến trúc, những giới hạn không quá dễ thấy trong project thường đáng giữ lại. Còn indent, thứ tự import và những việc tương tự thì giao cho formatting tool sẽ nhẹ đầu hơn.

Khi project còn nhỏ, cũng đừng vội dựng một hệ thống rule phức tạp. Trước tiên đặt một `CLAUDE.md` ngắn ở root, đủ dùng là được. Khi nội dung thực sự nhiều lên, hãy tách rule cục bộ sang `.claude/rules/` hoặc tài liệu riêng. Sau khi tách, dùng `/context` xem qua một lần, đừng nghĩ rằng đặt file đúng chỗ thì Claude chắc chắn đã đọc nó.

`CLAUDE.md` cũng không phải tài liệu viết xong rồi cất đi. Claude liên tục mắc cùng một lỗi thì bổ sung một rule; project thay đổi thì xóa rule cũ; những việc có thể bị chặn bằng test, Linter, Hook hoặc CI thì giao thẳng cho tool. File ngắn hơn, rule chính xác hơn thường hữu ích hơn việc bao quát mọi thứ.
